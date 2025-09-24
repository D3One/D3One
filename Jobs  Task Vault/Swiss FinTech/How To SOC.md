
# Build SOC on AWS with Open-Source (Wazuh, ELK/OpenSearch, TheHive, Cortex, MISP)

---

## 0) What you’ll build (high-level)

```
[AWS Accounts/Orgs]
  ├─ CloudTrail (all regions)  ─┐
  ├─ VPC Flow Logs ───────────┐ │
  ├─ ALB/NGINX/Apache logs ─┐ │ │
  ├─ GuardDuty findings ───┐│ │ │
  └─ Security Hub findings ││ │ │
                            ▼▼▼▼
                    [Ingestion Layer]
  Option A (all-OSS): Filebeat/Logstash → OpenSearch (or Elastic OSS)
  Option B (Wazuh-native): Wazuh AWS module → Wazuh Indexer (OpenSearch)
                            │
                            ▼
                   [Detection & SIEM]
                   Wazuh + Dashboards
                            │
                            ▼
                [IR & Threat Intelligence]
            TheHive (case mgmt) + Cortex (enrichment)
                          ↕
                         MISP (TI)
```

**Elements:**

* **Wazuh** = open-source SIEM/EDR with AWS modules, FIM, rules & SCA. It ships with its own **OpenSearch-based indexer** and dashboard (fully OSS). ([Wazuh Documentation][1])
* **TheHive** for case management (uses **Cassandra + Elasticsearch** for data/index). **Cortex** runs analyzers/responders and stores to Elasticsearch. ([StrangeBee Docs][2])
* **MISP** as your threat-intel hub; Cortex has MISP analyzers out of the box. ([MISP Threat Intelligence Platform][3])

---

## 1) Prerequisites

* **One AWS Organization** (prod + logging/security accounts recommended).
* **Ubuntu 22.04 LTS** EC2 for “SOC-Core” (TheHive+Cortex+ES/Cassandra) and **Ubuntu 22.04 LTS** EC2 for “Wazuh-Stack” (Wazuh manager + indexer + dashboard via Docker).
* Security groups: allow only SSH (from admin IP), HTTPS (443) from your office/VPN, and east-west only between SOC nodes.
* Registered DNS + TLS certs (ACM or Let’s Encrypt).

---

## 2) Turn on AWS security data sources (CLI)

### 2.1 CloudTrail (multi-region) → S3 (+ integrity + CW Logs)

```bash
# Create (or reuse) an S3 bucket for CloudTrail
aws s3api create-bucket --bucket org-cloudtrail-logs-<acct>-<region> --region <region> \
  --create-bucket-configuration LocationConstraint=<region>

# Create the trail (multi-region + log file validation)
aws cloudtrail create-trail \
  --name OrgTrail \
  --s3-bucket-name org-cloudtrail-logs-<acct>-<region> \
  --is-multi-region-trail \
  --enable-log-file-validation

# (Optional) Wire CloudWatch Logs for near-real-time
aws cloudtrail update-trail \
  --name OrgTrail \
  --cloud-watch-logs-log-group-arn arn:aws:logs:<region>:<acct>:log-group:CloudTrail-Org \
  --cloud-watch-logs-role-arn arn:aws:iam::<acct>:role/CloudTrailToCloudWatchRole

# Start logging
aws cloudtrail start-logging --name OrgTrail
```

CloudTrail multi-region trails + log file validation are documented here. ([AWS Documentation][4])

### 2.2 VPC Flow Logs → CloudWatch (or S3)

```bash
# Create Flow Logs for a VPC to CloudWatch Logs
aws ec2 create-flow-logs \
  --resource-type VPC --resource-ids vpc-1234567890abcdef0 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name VPCFlow-Prod \
  --deliver-logs-permission-arn arn:aws:iam::<acct>:role/FlowLogsToCWL \
  --max-aggregation-interval 60
```

CLI for `create-flow-logs` and examples with 1-minute aggregation are here. ([AWS Documentation][5])

### 2.3 GuardDuty + sample findings

```bash
# Enable GuardDuty in the region
aws guardduty create-detector --enable
# (capture the DetectorId from output)

# Generate sample findings for testing end-to-end
aws guardduty create-sample-findings --detector-id <DetectorId>
```

GuardDuty enablement and detector creation via CLI are covered here; sample findings are a handy test. ([AWS Documentation][6])

### 2.4 Security Hub (for posture + normalized findings)

Turn on **Security Hub** (and standards) organization-wide; later you can ingest findings into Wazuh as well.

* Set up via console or central config; multi-account enablement is documented here. ([AWS Documentation][7])

---

## 3) Option A (pure OSS logs path): Filebeat AWS module → OpenSearch/Elastic

You can pull **CloudTrail/CloudWatch/S3** directly using **Filebeat AWS module** and the **S3/SQS input**:

**Install Filebeat on your “Ingestion” host (same box as Wazuh indexer or separate):**

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.13.4-amd64.deb
sudo dpkg -i filebeat-8.13.4-amd64.deb

sudo filebeat modules enable aws
```

**Minimal `filebeat.yml` (S3→SNS→SQS pattern):**

```yaml
filebeat.inputs:
  - type: aws-s3
    queue_url: https://sqs.<region>.amazonaws.com/<acct>/<cloudtrail-sqs>
    expand_event_list_from_field: Records
    credential_profile_name: fb-aws

# (optional) CloudWatch metrics collection too
# module docs also support CloudWatch:
# https://www.elastic.co/docs/reference/integrations/aws/cloudwatch

output.opensearch:
  hosts: ["https://opensearch.yourdomain:9200"]
  username: "ingest"
  password: "${OPENSEARCH_PASS}"
  ssl.verification_mode: full
```

* AWS module & S3 input documentation (with SQS notifications) are here. ([Elastic][8])
* Direct CloudTrail/CloudWatch integration references (Elastic Integrations) are here. ([Elastic][9])

> **Tip:** for a 100% OSS search tier, use **OpenSearch** (fully Apache 2.0, and Wazuh’s indexer is OpenSearch-based). ([Wazuh Documentation][1])

---

## 4) Option B (simplest path): deploy **Wazuh Stack** via Docker (manager + indexer + dashboard)

**Why:** this gives you SIEM + index + dashboards in one go, plus native AWS modules.

### 4.1 Install Docker & compose, then run Wazuh containers

```bash
# On Ubuntu 22.04
sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
   https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
 | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER

# Wazuh compose
mkdir -p ~/wazuh && cd ~/wazuh
curl -sO https://packages.wazuh.com/4.x/docker-compose.yml
docker compose up -d
```

* Wazuh’s official Docker deployment (what those logs mean, timing for indexer, default creds) is documented here. ([Wazuh Documentation][10])

**Access:** `https://<WAZUH_HOST>:443` → Wazuh Dashboard.

> The Wazuh “Indexer” is **OpenSearch**; single- or multi-node install is covered in “Wazuh indexer step-by-step” (including cert generation). ([Wazuh Documentation][11])

---

## 5) Ingest AWS sources into **Wazuh**

Wazuh has first-class support for **CloudTrail**, **GuardDuty**, and **Security Hub**.

### 5.1 IAM policy for the Wazuh AWS module (minimal example)

Create a user/role with read access to your CloudTrail/GuardDuty/Security Hub S3 and/or API calls the module needs (see Wazuh docs for exact privileges per service). ([Wazuh Documentation][12])

### 5.2 Enable the AWS module in `ossec.conf` (on Wazuh manager)

```xml
<ossec_config>
  <cloud>
    <service name="aws">
      <!-- CloudTrail via S3/SQS -->
      <bucket type="cloudtrail">
        <name>org-cloudtrail-logs-...</name>
        <path>AWSLogs</path>
        <aws_profile>soc-profile</aws_profile>
        <sqs_name>cloudtrail-sqs</sqs_name>
        <trail_prefix></trail_prefix>
      </bucket>

      <!-- GuardDuty (S3 or API depending on setup) -->
      <service name="guardduty">
        <aws_profile>soc-profile</aws_profile>
        <region>us-east-1</region>
      </service>

      <!-- Security Hub (API) -->
      <service name="securityhub">
        <aws_profile>soc-profile</aws_profile>
        <region>us-east-1</region>
      </service>
    </service>
  </cloud>
</ossec_config>
```

* GuardDuty/CloudTrail/Security Hub integrations are documented here. ([Wazuh Documentation][12])

**Restart:**

```bash
sudo systemctl restart wazuh-manager
```

> **Troubleshooting:** Wazuh reads “new logs” based on last processed object key—know this when you replay data. Use `wazuh-logtest` for rules/decoders testing. ([Wazuh Documentation][13])

---

## 6) Deploy **TheHive** + **Cortex** (IR & enrichment)

> **Persistence backends:**
> • **TheHive 5** uses **Cassandra** for data and **Elasticsearch** for full-text indexing.
> • **Cortex** stores results in **Elasticsearch** (7.x API). ([StrangeBee Docs][2])

> **Compatibility tip:** Use **Elasticsearch 7.17 OSS** for TheHive/Cortex to avoid OpenSearch API edge cases. (TheHive docs reference ES; OpenSearch is often compatible but not guaranteed for all features.)

### 6.1 Install (Deb/RPM or containers)

* Official install docs for **Cortex** and analyzers. ([StrangeBee Docs][14])
* TheHive database/index configuration overview. ([StrangeBee Docs][2])

Below is a compact **docker-compose** to get you started (not hardened):

```yaml
version: "3.9"
services:
  es717:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.19
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ulimits: { memlock: { soft: -1, hard: -1 } }
    ports: ["9201:9200"]

  cassandra:
    image: cassandra:4.1
    environment: [ "HEAP_NEWSIZE=256M", "MAX_HEAP_SIZE=1G" ]
    ports: ["9042:9042"]

  thehive:
    image: strangebee/thehive:5
    depends_on: [ es717, cassandra ]
    ports: ["9000:9000"]
    volumes:
      - ./thehive/application.conf:/etc/thehive/application.conf:ro

  cortex:
    image: thehiveproject/cortex:3.1.7
    depends_on: [ es717 ]
    ports: ["9001:9001"]
    volumes:
      - ./cortex/application.conf:/etc/cortex/application.conf:ro
      - ./analyzers:/opt/Cortex-Analyzers
```

**`thehive/application.conf` (minimal):**

```hocon
db.janusgraph.storage.backend = cql
db.janusgraph.storage.hostname = "cassandra"
search.index.type = elasticsearch
search.index.hostname = ["es717"]
play.http.secret.key = "CHANGE-THIS"
```

**`cortex/application.conf` (points to ES):**

```hocon
search {
  index = cortex
  uri = "http://es717:9200"
}
```

**Start:**

```bash
docker compose up -d
```

---

## 7) Install **MISP** and wire it to Cortex (analyzers)

The easiest path is **MISP-dockerized**:

```bash
# On a dedicated small VM (or same host if testing)
git clone https://github.com/MISP/misp-docker.git
cd misp-docker
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

* MISP official docs + docker projects here. ([MISP Threat Intelligence Platform][3])

**Enable Cortex MISP analyzers (quick):**

```bash
# inside your analyzers folder
git clone https://github.com/TheHive-Project/Cortex-Analyzers.git
cd Cortex-Analyzers
# install per-analyzer deps
for I in $(find . -name requirements.txt); do sudo -H pip3 install -r "$I" || true; done
```

Then configure MISP URL & API key in Cortex UI for those analyzers. ([StrangeBee Docs][15])

---

## 8) Glue it together: alerts → cases → enrichment

### 8.1 Send Wazuh alerts to TheHive (simple webhook script)

Wazuh writes JSON alerts to `/var/ossec/logs/alerts/alerts.json`. A tiny Python forwarder can POST high-severity ones to TheHive’s API:

```python
# /usr/local/bin/wazuh_to_thehive.py
import json, requests, time
THEHIVE_URL="https://thehive.yourdomain:9000/api/v1/alert"
APIKEY="THEHIVE_API_KEY"

def post_alert(a):
    payload={
      "title": f"Wazuh: {a['rule']['description']}",
      "severity": a['rule']['level'],
      "tlp":2, "tags": ["Wazuh","AWS"],
      "source":"wazuh","type":"external",
      "description": json.dumps(a)[:8192]
    }
    r=requests.post(THEHIVE_URL, headers={"Authorization": f"Bearer {APIKEY}"}, json=payload, timeout=10)
    r.raise_for_status()

with open("/var/ossec/logs/alerts/alerts.json","r") as f:
    f.seek(0,2)
    while True:
        line = f.readline()
        if not line: time.sleep(0.2); continue
        try:
            a=json.loads(line)
            if a.get("rule",{}).get("level",0) >= 8:
                post_alert(a)
        except Exception as e:
            pass
```

**Systemd unit** to run it on boot, and you have an MVP pipeline.

* TheHive API “create alert/case” is documented here. ([StrangeBee Docs][16])

### 8.2 Run Cortex analyzers automatically from TheHive

In TheHive UI, create a **Responder/Analyzer** playbook that triggers on new alerts/cases to run **MISP**, **VirusTotal**, **urlscan**, etc.

* Cortex install/API docs and analyzer authoring are here. ([StrangeBee Docs][14])

---

## 9) Detection content you can paste today

### 9.1 “Console login without MFA” (Elastic KQL example)

```kql
event.dataset : "aws.cloudtrail" and event.action : "ConsoleLogin" and
aws.cloudtrail.user_identity.type : "IAMUser" and
aws.cloudtrail.response_elements.MFAAuthenticated : "false"
```

* CloudTrail parsing via Elastic AWS integration. ([Elastic][9])

### 9.2 Wazuh FIM for critical Linux paths

```xml
<syscheck>
  <directories check_all="yes">/etc/sudoers,/etc/sudoers.d,/etc/ssh</directories>
  <directories>/var/www</directories>
  <realtime>yes</realtime>
  <scan_on_start>yes</scan_on_start>
</syscheck>
```

* Wazuh FIM/SCA/Rootcheck references.

### 9.3 GuardDuty high-severity findings → case

Use Wazuh AWS/GuardDuty integration to ingest, and a small Wazuh rule threshold `level>=12` to forward to TheHive (script above). ([Wazuh Documentation][12])

---

## 10) Verify end-to-end (safe tests)

1. **Generate GuardDuty samples** (already shown) → confirm Wazuh alert appears and TheHive case is created. ([The Hidden Port][17])
2. **Toggle a Security Group to 0.0.0.0/0:22** in a test VPC → ensure CloudTrail → Wazuh rule fires. ([Wazuh Documentation][18])
3. **Touch `/etc/sudoers`** on a test box → see Wazuh FIM alert and case creation.

---

## 11) Hardening & scale notes (read before prod)

* **TLS everywhere** (reverse proxies or native TLS in ES/OpenSearch, Wazuh indexer certs per the “certificates creation” step). ([Wazuh Documentation][11])
* **IAM least-privilege** for Wazuh AWS module and S3/SQS read. (See each Wazuh AWS service page for exact perms.) ([Wazuh Documentation][12])
* **Security Hub** as continuous control monitoring; let it ingest GuardDuty too to unify posture. ([AWS Documentation][19])
* **Log delivery patterns:** For S3-based sources, prefer **S3→SNS→SQS** fan-out then pull with Filebeat or Wazuh AWS module. ([Elastic][20])
* **If you prefer managed search:** you can stream **CloudWatch Logs → Amazon OpenSearch Service** directly (managed), but this guide stays OSS-first. ([AWS Documentation][21])

---

## 12) Quick “Day-2” playbooks in TheHive

**Create case via API (curl skeleton):**

```bash
curl -X POST https://thehive.yourdomain:9000/api/v1/case \
  -H "Authorization: Bearer <APIKEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "title":"Suspicious IAM change",
    "severity":3,
    "tlp":2,
    "tags":["AWS","CloudTrail"],
    "description":"Auto-created from Wazuh alert ..."
  }'
```

* API references for TheHive are here. ([StrangeBee Docs][16])

**Run a Cortex analyzer via API** (e.g., on an IP from the alert) — see Cortex API guide. ([StrangeBee Docs][22])

---

## 13) Appendix: Minimal NGINX reverse-proxy (TLS) in front of TheHive/Cortex

```nginx
server {
  listen 443 ssl http2;
  server_name soc.yourdomain;

  ssl_certificate     /etc/letsencrypt/live/soc/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/soc/privkey.pem;

  location /thehive/ { proxy_pass http://127.0.0.1:9000/; }
  location /cortex/  { proxy_pass http://127.0.0.1:9001/; }
}
```

(Place TheHive at `/thehive/`, Cortex at `/cortex/`, protect with IP allow-lists and SSO if possible.)

---

## 14) Common pitfalls & fixes

* **TheHive + OpenSearch?** Many features work, but stick to **Elasticsearch 7.17 OSS** for full compatibility per TheHive docs that reference ES explicitly. ([StrangeBee Docs][2])
* **Wazuh not pulling new S3 objects?** It tracks the last processed key; rotate prefixes or purge the state DB if you must replay (see “considerations”). ([Wazuh Documentation][13])
* **Analyzers failing in Cortex?** Ensure each analyzer’s `requirements.txt` is installed and API keys are set. ([StrangeBee Docs][15])

---

## 15) Where to read more (authoritative)

* **Wazuh AWS integrations** (CloudTrail, GuardDuty, Security Hub), rules/decoders/testing. ([Wazuh Documentation][18])
* **Wazuh Docker & Indexer** (OpenSearch) deploy docs. ([Wazuh Documentation][10])
* **TheHive + Cortex** install, DB/index config, API & analyzers. ([StrangeBee Docs][2])
* **MISP docs & Docker variants.** ([MISP Threat Intelligence Platform][3])
* **Elastic/Beats AWS integrations** (CloudTrail/CloudWatch) and S3/SQS ingestion. ([Elastic][9])
* **AWS references** for CloudTrail, VPC Flow Logs, GuardDuty, Security Hub. ([AWS Documentation][23])

---

[1]: https://documentation.wazuh.com/current/integrations-guide/opensearch/index.html?utm_source=chatgpt.com "OpenSearch integration"
[2]: https://docs.strangebee.com/thehive/configuration/database/?utm_source=chatgpt.com "Database & Index Configuration - TheHive 5 Documentation"
[3]: https://www.misp-project.org/documentation/?utm_source=chatgpt.com "MISP Documentation and Support"
[4]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail-by-using-the-aws-cli-create-trail.html?utm_source=chatgpt.com "Using the create-trail command to create a trail"
[5]: https://docs.aws.amazon.com/cli/latest/reference/ec2/create-flow-logs.html?utm_source=chatgpt.com "create-flow-logs — AWS CLI 2.30.6 Command Reference"
[6]: https://docs.aws.amazon.com/cli/latest/reference/guardduty/create-detector.html?utm_source=chatgpt.com "create-detector — AWS CLI 2.28.4 Command Reference"
[7]: https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html?utm_source=chatgpt.com "Enabling Security Hub CSPM - AWS Documentation"
[8]: https://www.elastic.co/docs/reference/beats/filebeat/filebeat-module-aws?utm_source=chatgpt.com "AWS module | Beats"
[9]: https://www.elastic.co/docs/reference/integrations/aws/cloudtrail?utm_source=chatgpt.com "AWS CloudTrail | Elastic integrations"
[10]: https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html?utm_source=chatgpt.com "Wazuh Docker deployment"
[11]: https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/step-by-step.html?utm_source=chatgpt.com "Installing the Wazuh indexer step by step"
[12]: https://documentation.wazuh.com/current/cloud-security/amazon/services/supported-services/guardduty.html?utm_source=chatgpt.com "Amazon GuardDuty - Supported services"
[13]: https://documentation.wazuh.com/current/cloud-security/amazon/services/prerequisites/considerations.html?utm_source=chatgpt.com "Considerations for the Wazuh module for AWS configuration"
[14]: https://docs.strangebee.com/cortex/installation-and-configuration/?utm_source=chatgpt.com "Installation and Configuration Guides"
[15]: https://docs.strangebee.com/cortex/installation-and-configuration/analyzers-responders/?utm_source=chatgpt.com "Analyzers & Responders - TheHive 5 Documentation"
[16]: https://docs.strangebee.com/thehive/api-docs?utm_source=chatgpt.com "TheHive 5 API Documentation"
[17]: https://thehiddenport.dev/posts/aws-guardduty-setup/?utm_source=chatgpt.com "Getting Started with Amazon GuardDuty: Setup, Findings, and SIEM ..."
[18]: https://documentation.wazuh.com/current/cloud-security/amazon/services/supported-services/cloudtrail.html?utm_source=chatgpt.com "AWS CloudTrail - Supported services"
[19]: https://docs.aws.amazon.com/guardduty/latest/ug/securityhub-integration.html?utm_source=chatgpt.com "Integrating with AWS Security Hub - Amazon GuardDuty"
[20]: https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-aws-s3?utm_source=chatgpt.com "AWS S3 input | Beats"
[21]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_OpenSearch_Stream.html?utm_source=chatgpt.com "Streaming CloudWatch Logs data to Amazon OpenSearch ..."
[22]: https://docs.strangebee.com/cortex/api/api-guide/?utm_source=chatgpt.com "API Guide - TheHive 5 Documentation - StrangeBee"
[23]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail-by-using-the-aws-cli.html?utm_source=chatgpt.com "Creating, updating, and managing trails with the AWS CLI"
