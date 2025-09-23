
## 🛡️ Introduction to the Google Cloud Certified Professional Cloud Security Engineer Exam

The **Professional Cloud Security Engineer** certification validates a professional's ability to design, develop, and manage secure solutions and infrastructure on the Google Cloud Platform (GCP). This individual ensures that identity and access management, data protection, network security, logging/monitoring, and compliance are effectively implemented using Google Cloud technologies. As of 2021, this exam was a critical credential for professionals aiming to secure cloud environments in an era defined by digital transformation, the shift to microservices, and the increasing value of data security.

---

### 📚 Sample Exam Questions (2021 Context)

Here are 15 questions reflecting the topics and style of the exam during the period you took it.

#### 🔍 Theoretical Questions

**1. What is the primary purpose of an Organization Policy in Google Cloud?**
*   A. To manage user permissions for specific resources.
*   B. To define centralized rules and constraints for all resources within the organization's hierarchy.
*   C. To monitor regulatory compliance.
*   D. To encrypt data on virtual machines.

**Answer: B.** Organization Policy imposes constraints at the organization, folder, or project level (e.g., disabling the creation of external IPs), whereas IAM (A) manages "who can do what" on which resource.

**2. The Google Cloud Identity-Aware Proxy (IAP) service provides secure access to applications by...**
*   A. Validating only the source IP address of a request.
*   B. Authenticating the user and applying context-aware access policies, eliminating the need for a VPN.
*   C. Automatically encrypting all data in transit using Cloud KMS keys.
*   D. Working only with services running on Google Kubernetes Engine (GKE).

**Answer: B.** IAP is a key service for implementing a Zero-Trust model, allowing access to applications (on VMs, GKE, or App Engine) based on user identity and context, not network location.

**3. Which of the following is the best method for auditing actions and access to resources within a GCP project?**
*   A. Rely solely on individual service logs in Stackdriver Logging.
*   B. Configure the export of audit logs from Cloud Audit Logs (Data Access and Admin Activity) to BigQuery or Cloud Storage for centralized analysis.
*   C. Run a script regularly that lists all resources using the gcloud CLI.
*   D. Review notifications in the Google Cloud Console dashboard.

**Answer: B.** Cloud Audit Logs is the primary auditing service in GCP. Exporting logs to long-term storage is a recommended practice for preservation and deep analysis.

**4. The Principle of Least Privilege in the context of GCP IAM means...**
*   A. Granting all users in a project the `Viewer` role by default.
*   B. Assigning roles that contain only the permissions necessary to perform specific tasks.
*   C. Using only primitive roles like `Owner`, `Editor`, and `Viewer`.
*   D. Mandating Multi-Factor Authentication (MFA) for all users.

**Answer: B.** This is a fundamental security principle. In GCP, it's best practiced by using predefined or custom roles with minimal required permissions, avoiding broad primitive roles.

**5. Which Google Cloud service is designed to automatically discover and classify sensitive data (like credit card or passport numbers) in datasets?**
*   A. Cloud Key Management Service (KMS)
*   B. Cloud Data Loss Prevention (DLP) API
*   C. Cloud Security Scanner
*   D. Cloud Data Catalog

**Answer: B.** The Cloud DLP API is a powerful tool for scanning, masking, and de-identifying sensitive data.

#### ⚙️ Practical Questions (CLI, Configurations)

**6. You want to verify if the restriction for sharing resources only within your domain (Domain Restricted Sharing) is enabled for your organization. Which gcloud CLI command would you run?**
*   A. `gcloud organizations describe ORGANIZATION_ID`
*   B. `gcloud resource-manager org-policies list ORGANIZATION_ID`
*   C. `gcloud projects get-iam-policy PROJECT_ID`
*   D. `gcloud iam service-accounts list`

**Answer: B.** This command lists organization policies. To check the specific `iam.allowedPolicyMemberDomains` constraint, more detailed commands can be used.

**7. You need to ensure that Cloud KMS encryption keys are automatically rotated every 90 days. What do you do?**
*   A. Manually create a new key version every quarter using `gcloud kms keys versions create`.
*   B. Configure the key rotation policy when creating or updating the key, setting the period to `7776000s` (90 days).
*   C. Create a Cloud Scheduler job that triggers a Cloud Function to rotate the key.
*   D. Open a support ticket with Google Cloud to request rotation setup.

**Answer: B.** Cloud KMS supports automatic key rotation. The period is specified in seconds. `7776000 seconds = 90 days`.

**8. Which command lists all service accounts in a project?**
*   A. `gcloud iam service-accounts list`
*   B. `gcloud projects get-iam-policy PROJECT_ID`
*   C. `gcloud kms keys list --keyring=KEY_RING_NAME --location=global`
*   D. `gcloud compute instances list`

**Answer: A.** This is the standard command for listing service accounts.

**9. You find a service account in your project's IAM policy with the `Editor` role. Is this a good practice?**
*   A. Yes, this is standard practice.
*   B. No, service accounts should be assigned roles with the least privileges necessary for their task.
*   C. It doesn't matter because service accounts are not users.
*   D. Yes, but only if it's the default Compute Engine service account.

**Answer: B.** Assigning broad roles like `Editor` or `Owner` to service accounts creates significant risk if the key is compromised. The principle of least privilege should always be applied.

**10. Which Google Cloud Organization Policy constraint can be used to prevent the creation of publicly accessible objects?**
*   A. `constraints/iam.allowedPolicyMemberDomains`
*   B. `constraints/storage.publicAccessPrevention`
*   C. `constraints/compute.restrictExternalIpAddresses`
*   D. `constraints/storage.uniformBucketLevelAccess`

**Answer: B.** The `storage.publicAccessPrevention` policy enforces the prevention of public access to data in Cloud Storage.

**11. For secure access to private Compute Engine instances without using bastion hosts or opening SSH ports, it is recommended to use...**
*   A. Cloud VPN
*   B. Identity-Aware Proxy (IAP) for TCP Forwarding
*   C. Cloud Interconnect
*   D. Configuring a firewall rule allowing `0.0.0.0/0` on port 22

**Answer: B.** IAP TCP Forwarding allows you to establish SSH and RDP connections to VMs through Google's centralized proxy, without the need for public IPs or open firewall ports.

**12. Which service is responsible for creating and managing cryptographic keys in GCP?**
*   A. Cloud HSM
*   B. Cloud Key Management Service (KMS)
*   C. Cloud Data Loss Prevention (DLP)
*   D. Cloud Identity and Access Management (IAM)

**Answer: B.** Cloud KMS is the centralized service for managing encryption keys.

**13. Which command allows you to check if user-managed keys have been created for a service account?**
*   A. `gcloud iam service-accounts keys list --iam-account=SERVICE_ACCOUNT --managed-by=user`
*   B. `gcloud iam service-accounts describe SERVICE_ACCOUNT`
*   C. `gcloud kms keys list --keyring=KEY_RING_NAME`
*   D. `gcloud projects get-iam-policy PROJECT_ID`

**Answer: A.** This command lists all user-managed keys for the specified service account. Best practice is to avoid these keys and use Google-managed alternatives where possible.

**14. Which service is used to scan for vulnerabilities in web applications deployed on App Engine, GKE, or Compute Engine VMs?**
*   A. Cloud Security Command Center (SCC)
*   B. Cloud Security Scanner
*   C. Web Security Scanner (now part of Cloud Security Scanner)
*   D. Google Chronicle

**Answer: B and C.** As of 2021, the service was named **Web Security Scanner** and was designed for automatically finding vulnerabilities like XSS and mixed content in web applications.

**15. What is the recommended way to manage attribute-based access control (e.g., based on device security or user location)?**
*   A. Using Context-Aware Access as part of BeyondCorp Enterprise.
*   B. Creating complex custom IAM roles.
*   C. Configuring VPC firewall rules.
*   D. Using Organization Policies.

**Answer: A.** Context-Aware Access allows defining granular access policies to applications and resources based on attributes such as IP address, device certificate, or device state, which is part of the BeyondCorp model.

---

### 📖 Recommended Preparation Resources

*   **Official Exam Guide:** The primary document from Google containing the complete list of topics.
*   **Google Cloud Documentation:** The exhaustive source of information for all services, especially sections on IAM, Cloud KMS, VPC, Audit Logs, and Organization Policies.
*   **Google Cloud Security Best Practices:** Official whitepapers and guides on security best practices.
*   **Coursera Courses:** The "Preparing for Google Cloud Certification: Cloud Security Engineer" course series.
*   **Qwiklabs / Google Cloud Skills Boost:** The platform for gaining hands-on experience with GCP through interactive labs.

### 💡 Conclusion

This list of questions, compiled by **Ivan Piskunov (c) 2021**, is based on the exam topics relevant to the Professional Cloud Security Engineer exam at that time.

**Important Note for 2025:** The exam blueprint, services, and Google Cloud best practices are constantly evolving. For instance, the Web Security Scanner service is now named Cloud Security Scanner, and BeyondCorp capabilities have expanded. Therefore, **the information in this article may be outdated** and might not reflect the current exam requirements or Google's recommendations. However, the understanding of fundamental concepts like least privilege, encryption, monitoring, and network controls remains perpetually valuable.
