
# 🔒 MySQL Security Auditing in Practice: A Field Guide for Financial Data Stores

*Author: Ivan Piskunov 

Source material compiled from official docs, books, and vendor references. 

The goal: enable **provable**, **tamper-resistant**, and **actionable** audit logs around admin/privileged activity that could alter critical financial data.*

---

## 🧭 Preamble — what we’re doing and why

**MySQL** (5.7 → 8.0 era) gives you multiple levers for security monitoring and auditing: the **error log**, **general** and **slow** query logs, the **binary log** (binlog), **Performance Schema** + **sys** views, and—on Enterprise or Percona builds—**audit plugins** that produce policy-driven activity logs (logins, DDL/DML, etc.). We’ll wire these up with a bias toward **integrity of the audit trail**, **operator ergonomics**, and **SIEM-friendly formats**. ([Oracle Documentation][1])

> **TL;DR:**
>
> * Use **MySQL Enterprise Audit** (or **Percona Audit Log**) for canonical activity auditing; prefer **JSON** output and **log encryption**. ([MySQL][2])
> * Keep **general log** off by default; rely on **audit** + **Performance Schema/sys** + **slow log** for observability with sane overhead. ([MySQL][3])
> * Run **binlog in ROW** format with sensible retention; it’s not an audit log, but it’s gold for forensics/replication. ([MySQL][4])

---

## 🧩 Building blocks (what exists)

* **Error log:** startup/shutdown/events; in 8.0 you can emit **JSON** and/or ship to **syslog** via `log_error_services` components. Great for **tamper-evident** pipelines. ([Oracle Documentation][5])
* **General query log:** every statement + connects/disconnects; **high overhead** and potentially sensitive payloads—use sparingly or during incident response. ([MySQL][6])
* **Slow query log:** statements exceeding `long_query_time` (plus options like `log_queries_not_using_indexes`). Low noise; good for tuning and anomaly detection. ([MySQL][7])
* **Binary log (binlog):** data-change events for **replication/point-in-time recovery**; default in 8.0 is enabled and **ROW** is recommended/default—useful for reconstruction but doesn’t log `SELECT`. ([MySQL][8])
* **Performance Schema + sys schema:** structured telemetry on statements/users/latency; **sys** views like `statement_analysis` & `user_summary_by_statement_type` are auditor-friendly. ([MySQL][9])
* **Audit plugins:**

  * **MySQL Enterprise Audit** (`audit_log` plugin) — policy-based, filterable, logins/queries/DDL; supports **encryption**, **rotation**, **include/exclude**, and **JSON** logs. ([MySQL][2])
  * **Percona Audit Log** — community alternative with similar scope; XML/JSON depending on version. ([docs.percona.com][10])

---

## 🛡️ Hardening prerequisites (security posture that supports auditing)

**Enforce TLS** for clients and reject plaintext:

```ini
# /etc/my.cnf
[mysqld]
require_secure_transport = ON     # force TLS or local socket/shared mem
```

*Why:* blocks non-TLS TCP; failed attempts are explicit (`ER_SECURE_TRANSPORT_REQUIRED`). Expect slight CPU/network cost. ([MySQL][11])

**Modern auth & password policy:**

```sql
-- Default in 8.0 is caching_sha2_password; keep it unless compat demands otherwise
SELECT @@authentication_policy;  -- confirm policy stack (factor 1 default is caching_sha2_password)

-- Enable password validation component
INSTALL COMPONENT 'file://component_validate_password';
-- Tune policy (examples)
SET PERSIST validate_password.length = 14;
SET PERSIST validate_password.mixed_case_count = 1;
SET PERSIST validate_password.number_count = 1;
SET PERSIST validate_password.special_char_count = 1;
```

*Why:* **SHA-2** based auth by default in 8.0; use `validate_password` to enforce strong creds. ([MySQL][12])

---

## 📝 Error logging (structured, centralizable)

**Goal:** machine-readable, tamper-evident error pipeline (JSON + syslog).

```sql
-- Enable JSON error sink and (optionally) syslog sink
INSTALL COMPONENT 'file://component_log_sink_json';
INSTALL COMPONENT 'file://component_log_sink_syseventlog';

-- Route error events through internal filter -> JSON file and/or syslog
SET PERSIST log_error_services =
  'log_filter_internal; log_sink_json; log_sink_syseventlog';
```

*Why:* **Component-based** error logging lets you write to multiple sinks; JSON is SIEM-friendly, syslog simplifies centralization. ([Oracle Documentation][13])

---

## 🐌 Slow query log (low-noise visibility)

```ini
# /etc/my.cnf
[mysqld]
slow_query_log         = ON
long_query_time        = 0.5        # tighten for fintech workloads
log_queries_not_using_indexes = OFF # keep noise low unless investigating
log_output             = FILE       # or TABLE if you want SQL access
```

*Why:* captures **problem statements** without the firehose. Choose **FILE** (lighter) or **TABLE** (`mysql.slow_log`). ([MySQL][7])

---

## 📓 General query log (only when you really need it)

```sql
-- Temporary enablement during incident response
SET GLOBAL log_output = 'FILE';
SET GLOBAL general_log_file = '/var/log/mysql/general-%Y%m%d.log';
SET GLOBAL general_log = 'ON';
-- ... reproduce issue ...
SET GLOBAL general_log = 'OFF';
```

*Why:* logs **everything** (includes sensitive literals). Keep off in steady state. ([MySQL][6])

---

## 🔁 Binary log for forensics (not an audit, but essential)

```ini
[mysqld]
# 8.0: binlog is ON by default; make the format explicit
binlog_format = ROW
binlog_row_image = FULL           # fuller change context for forensics
binlog_expire_logs_seconds = 604800  # 7 days (tune per storage/SLA)
max_binlog_size = 256M
```

*Why:* **ROW** best reflects actual row changes (default in 8.0). Longer retention aids investigations. **Note:** binlog does **not** log `SELECT`. ([MySQL][4])

> 🔎 *Red flag to monitor:*
> Some admins try `SET SESSION sql_log_bin=OFF;` to keep changes out of the binlog. Alert on this and require justification/multi-party control. ([MySQL][14])

---

## 🧰 Performance Schema + sys = auditor dashboards

**Enable statement history and use sys views for per-user summaries:**

```sql
-- Ensure statement consumers are on (defaults are usually fine)
UPDATE performance_schema.setup_consumers
  SET ENABLED='YES'
  WHERE NAME IN ('events_statements_current','events_statements_history','events_statements_history_long');

-- Who is doing what, by statement type:
SELECT * FROM sys.user_summary_by_statement_type ORDER BY total_latency DESC LIMIT 50;

-- Top normalized statements:
SELECT * FROM sys.statement_analysis ORDER BY total_latency DESC LIMIT 50;
```

*Why:* **Low-overhead** visibility into behavior patterns; great complement to audit logs. ([MySQL][15])

---

## 🧿 MySQL Enterprise Audit (preferred, commercial)

> **License note:** The following requires **MySQL Enterprise Edition**. Community equivalents below.

### 1) Install & lock the plugin

```sql
-- Install plugin and persistent tables (8.0+)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';
-- Optional: force + protect from unload at startup
-- my.cnf: --audit-log=FORCE_PLUS_PERMANENT
```

*Why:* Loads the `audit_log` plugin; `FORCE_PLUS_PERMANENT` prevents removal online. ([MySQL][2])

### 2) Choose format, rotation, encryption

```ini
[mysqld]
audit_log_format = JSON                     # machine-readable
audit_log_file   = /var/log/mysql/audit.json
audit_log_strategy = ASYNCHRONOUS           # lower latency; SYNC for strictness
audit_log_rotate_on_size = 500M
audit_log_max_size       = 5G               # prune rotated JSON logs at 5G total
audit_log_encryption = AES                  # at-rest encryption (Enterprise)
```

*Why:* JSON + rotation + pruning = production-grade. **Encryption** uses the **keyring** (enable one). ([MySQL][2])

### 3) Enable key management (required for encrypted audit logs)

```ini
[mysqld]
early-plugin-load = keyring_file.so
keyring_file_data = /var/lib/mysql-keyring/keyring
```

*Why:* Audit log encryption keys are stored in **keyring**; plugin will auto-generate on first run. ([MySQL][16])

### 4) Define filters: log admins fully, others scoped

*Modern (filter tables) approach—preferred over legacy include/exclude variables.*

```sql
-- Create a filter that logs everything (connections, DDL, DML) with full status
INSERT INTO mysql.audit_log_filter (NAME, FILTER)
VALUES ('log_all', JSON_OBJECT());

-- Assign filter to sensitive accounts (DBAs, app superusers)
INSERT INTO mysql.audit_log_user (USER, HOST, FILTERNAME)
VALUES ('dba','%', 'log_all'),
       ('app_admin','%', 'log_all');

-- Optionally, a lighter filter for read-only accounts
INSERT INTO mysql.audit_log_filter (NAME, FILTER)
VALUES ('log_logins_only', JSON_OBJECT('class','connection'));

INSERT INTO mysql.audit_log_user (USER, HOST, FILTERNAME)
VALUES ('reporter','%', 'log_logins_only');
```

*Why:* **Filter tables** (`audit_log_filter`, `audit_log_user`) allow granular, JSON-based policy per account; legacy variables are deprecated. ([MySQL][2])

### 5) Verify and operate

```sql
SHOW VARIABLES LIKE 'audit_log%';   -- confirm effective parameters
SELECT audit_log_encryption_password_get();  -- keyring integration check
```

*Why:* Sanity check; use **bookmark/read** functions for controlled retrieval. ([MySQL][2])

---

## 🧪 Community alternative — Percona Audit Log (if Enterprise not available)

```ini
[mysqld]
plugin-load-add = audit_log.so
audit_log_policy = ALL
audit_log_file   = /var/log/mysql/audit.log
# Depending on version, format may be XML (5.7) or JSON (8.0 builds)
```

*Why:* Tracks connections & queries; verify format support for your Percona version. ([docs.percona.com][10])

---

## 🧷 Table-level change journals (surgical capture for key tables)

For **critical financial tables**, add **immutable “ledger” rows** via triggers (complements audit logs; works on Community/Enterprise):

```sql
CREATE TABLE audit_ledger (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  ts TIMESTAMP(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  actor VARCHAR(128) NOT NULL,
  host VARCHAR(255) NOT NULL,
  action ENUM('INSERT','UPDATE','DELETE') NOT NULL,
  tbl VARCHAR(128) NOT NULL,
  pk  JSON NOT NULL,
  old JSON NULL,
  new JSON NULL
) ENGINE=InnoDB;

DELIMITER //
CREATE TRIGGER trx_bu BEFORE UPDATE ON transactions
FOR EACH ROW
BEGIN
  INSERT INTO audit_ledger(actor,host,action,tbl,pk,old,new)
  VALUES(CURRENT_USER(), @@hostname, 'UPDATE', 'transactions',
         JSON_OBJECT('id', OLD.id),
         JSON_OBJECT('amount', OLD.amount, 'status', OLD.status),
         JSON_OBJECT('amount', NEW.amount, 'status', NEW.status));
END//
DELIMITER ;
```

*Why:* **Defense in depth**: guarantees per-row diffs even if someone tries to suppress higher-level logging. (Mind write-amplification.)

---

## 🧑‍💻 Operational guidance & pitfalls

* **Protect the audit trail:** lock file perms to the **mysqld** user, and **ship** logs off-host (syslog/agent). JSON sinks simplify ingestion. ([Oracle Documentation][13])
* **Monitor for tampering:** alert on `SET SESSION sql_log_bin = OFF`, plugin unload attempts, or disabling audit (`audit_log_disable`). These operations require elevated privileges—treat them as **break-glass** events. ([MySQL][14])
* **Keep general log off** by default; if you must enable, use **short windows** and scrub PII in analytics. ([MySQL][6])
* **Tune slow log** conservatively (`0.5s` or lower) in fintech workloads; raise temporarily during bulk loads. ([MySQL][7])
* **Use ROW binlog** with **FULL** row image when the forensic story matters more than storage. ([MySQL][17])
* **Lean on Performance Schema/sys** for “who did what, how often, how slow”. The `sys` views are designed to be human-readable; `x$` are raw and better for tooling. ([MySQL][18])

---

## 🧾 Quick reference — commands & config (copy/paste ready)

**my.cnf (core)**

```ini
[mysqld]
# Transport & auth
require_secure_transport = ON
# Logging (error)
log_error = /var/log/mysql/error.log
# Enable JSON + syslog via components at runtime (see SQL section)

# Slow log
slow_query_log = ON
long_query_time = 0.5
log_output = FILE

# Binlog (forensics/replication)
binlog_format = ROW
binlog_row_image = FULL
binlog_expire_logs_seconds = 604800
max_binlog_size = 256M

# Enterprise Audit (if licensed)
audit_log_format = JSON
audit_log_file = /var/log/mysql/audit.json
audit_log_strategy = ASYNCHRONOUS
audit_log_rotate_on_size = 500M
audit_log_max_size = 5G
audit_log_encryption = AES
```

**One-time SQL (components & audit)**

```sql
-- Error log sinks
INSTALL COMPONENT 'file://component_log_sink_json';
INSTALL COMPONENT 'file://component_log_sink_syseventlog';
SET PERSIST log_error_services =
  'log_filter_internal; log_sink_json; log_sink_syseventlog';

-- Enterprise Audit (if licensed)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';
INSERT INTO mysql.audit_log_filter (NAME, FILTER) VALUES ('log_all', JSON_OBJECT());
INSERT INTO mysql.audit_log_user (USER,HOST,FILTERNAME)
VALUES ('dba','%','log_all'),('app_admin','%','log_all');
```

(Each directive is commented/explained in its section above.)

---

## 📚 Official docs & solid prep material

* **MySQL Enterprise Audit**: features, install, filters, encryption/rotation, variables. ([MySQL][2])
* **Audit logging config & encryption (keyring)**: enabling AES, password management in keyring. ([MySQL][19])
* **General log / Slow log / Destinations (`log_output`)**: enable, file vs table, runtime control. ([MySQL][6])
* **Error log (JSON/syslog via components)**: `log_error_services`, sinks/filters. ([Oracle Documentation][13])
* **Binary log**: purpose, formats (`ROW`), defaults in 8.0, retention. ([MySQL][20])
* **Performance Schema & sys schema**: statement history tables & auditor views. ([MySQL][15])
* **Percona Audit Log Plugin** (community alternative). ([docs.percona.com][10])

**Books / further reading:**

* *MySQL 8.0 Reference Manual* (full PDF). ([downloads.mysql.com][21])
* *MySQL 8.0 Secure Deployment Guide* (hardening & auth/TLS). ([downloads.mysql.com][22])

---

## ✅ Final recommendations (bank-grade)

1. **Use Enterprise Audit** (or Percona) with **JSON + AES** and **keyring**; filter **DBA and app\_admin** accounts to **log everything**. ([MySQL][2])
2. **Ship logs** off-box (syslog) and **rotate** aggressively; protect file perms. ([MySQL][23])
3. Keep **general log OFF**; rely on **audit + slow log + sys views** daily; enable general log **only during IR**. ([MySQL][6])
4. **ROW binlog** with **FULL** image + retention that meets forensic SLA. ([MySQL][17])
5. **Alert** on **sql\_log\_bin OFF**, plugin unload, or audit disable attempts; treat as **break-glass**. ([MySQL][14])

[1]: https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/index.html?utm_source=chatgpt.com "MySQL 8.0 Reference Manual"
[2]: https://dev.mysql.com/doc/refman/8.4/en/audit-log-reference.html "MySQL :: MySQL 8.4 Reference Manual :: 8.4.5.11 Audit Log Reference"
[3]: https://dev.mysql.com/doc/refman/8.4/en/log-destinations.html?utm_source=chatgpt.com "7.4.1 Selecting General Query Log and Slow ..."
[4]: https://dev.mysql.com/doc/refman/8.3/en/binary-log-setting.html?utm_source=chatgpt.com "7.4.4.2 Setting The Binary Log Format"
[5]: https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/error-log-configuration.html?utm_source=chatgpt.com "7.4.2.1 Error Log Configuration"
[6]: https://dev.mysql.com/doc/refman/8.4/en/query-log.html?utm_source=chatgpt.com "7.4.3 The General Query Log"
[7]: https://dev.mysql.com/doc/en/slow-query-log.html?utm_source=chatgpt.com "7.4.5 The Slow Query Log"
[8]: https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-3.html?utm_source=chatgpt.com "Changes in MySQL 8.0.3 (2017-09-21, Release Candidate)"
[9]: https://dev.mysql.com/doc/mysql-perfschema-excerpt/8.0/en/?utm_source=chatgpt.com "MySQL Performance Schema"
[10]: https://docs.percona.com/percona-server/8.0/audit-log-plugin.html?utm_source=chatgpt.com "Audit log plugin - Percona Server for MySQL"
[11]: https://dev.mysql.com/doc/en/using-encrypted-connections.html?utm_source=chatgpt.com "8.3.1 Configuring MySQL to Use Encrypted Connections"
[12]: https://dev.mysql.com/doc/refman/en/caching-sha2-pluggable-authentication.html?utm_source=chatgpt.com "8.4.1.2 Caching SHA-2 Pluggable Authentication"
[13]: https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/error-log-json.html?utm_source=chatgpt.com "7.4.2.7 Error Logging in JSON Format"
[14]: https://dev.mysql.com/doc/refman/8.1/en/set-sql-log-bin.html?utm_source=chatgpt.com "15.4.1.3 SET sql_log_bin Statement"
[15]: https://dev.mysql.com/doc/mysql-perfschema-excerpt/8.0/en/performance-schema-events-statements-history-long-table.html?utm_source=chatgpt.com "10.6.3 The events_statements_history_long Table"
[16]: https://dev.mysql.com/doc/mysql-security-excerpt/8.0/en/audit-log-logging-configuration.html?utm_source=chatgpt.com "6.5.5 Configuring Audit Logging Characteristics"
[17]: https://dev.mysql.com/doc/refman/8.4/en/binary-log-formats.html?utm_source=chatgpt.com "7.4.4.1 Binary Logging Formats"
[18]: https://dev.mysql.com/doc/refman/en/sys-statement-analysis.html?utm_source=chatgpt.com "30.4.3.35 The statement_analysis and x$ ..."
[19]: https://dev.mysql.com/doc/refman/8.4/en/audit-log-logging-configuration.html?utm_source=chatgpt.com "8.4.5.5 Configuring Audit Logging Characteristics"
[20]: https://dev.mysql.com/doc/en/binary-log.html?utm_source=chatgpt.com "MySQL 8.4 Reference Manual :: 7.4.4 The Binary Log"
[21]: https://downloads.mysql.com/docs/refman-8.0-en.pdf?utm_source=chatgpt.com "MySQL 8.0 Reference Manual - Including MySQL NDB ..."
[22]: https://downloads.mysql.com/docs/mysql-secure-deployment-guide-8.0-en.pdf?utm_source=chatgpt.com "MySQL 8.0 Secure Deployment Guide"
[23]: https://dev.mysql.com/doc/refman/8.2/en/error-log-syslog.html?utm_source=chatgpt.com "7.4.2.8 Error Logging to the System Log"
