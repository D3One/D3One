
## 🛡️ Introduction to the AWS Certified Security - Specialty (SCS-C01) Exam

The **AWS Certified Security - Specialty (SCS-C01)** certification validates advanced technical skills and experience in securing the AWS platform. It is designed for professionals with a background in security who can demonstrate the ability to implement data protection mechanisms, secure internet protocols, and manage security operations on AWS.

This certification gained significant market value due to the accelerated shift to cloud computing, the growth of the "geek economy," and the widespread adoption of microservices. Companies developing software needed experts who could ensure robust security postures in dynamic cloud environments, making this certification a key differentiator for professionals.

**Exam Overview (circa 2020-2021):**
*   **Format:** 65 multiple-choice or multiple-response questions.
*   **Duration:** 170 minutes.
*   **Passing Score:** 750 out of 1000 points.

---

### 📚 Exam Topic Breakdown & Sample Questions

The following questions are a mix of theoretical knowledge and practical application, reflecting the style and content of the SCS-C01 exam during the 2020-2021 period.

#### 🔐 Foundational Security Services (Theoretical Focus)

**1. A company's IT audit department needs a list of all resources defined in their AWS account. What is the EASIEST way to achieve this?**
*   A. Create a PowerShell script using the AWS CLI to query for all resources in all regions and store the results in an S3 bucket.
*   B. Use AWS CloudTrail to get the list of all resources.
*   C. Use AWS Config to inventory the resources.
*   D. Use AWS Trusted Advisor under the "Security" category.

**Answer: C.** AWS Config is designed for resource inventory and configuration history. While a script could work, AWS Config is the managed service that provides this capability without the need for custom scripting.

**2. Which of the following objectives are achieved by implementing an IPsec tunnel between an on-premises network and an Amazon VPC? (Choose 3)**
*   A. End-to-end protection of data in transit.
*   B. Data encryption across the internet.
*   C. Peer identity authentication between VPN gateway and customer gateway.
*   D. Data integrity protection across the internet.
*   E. End-to-end Identity authentication.

**Answer: B, C, D.** An IPsec VPN provides encryption, peer authentication, and data integrity for the connection over the internet. It does not provide end-to-end protection or authentication, which would be the responsibility of the applications communicating over the tunnel.

**3. What is the primary difference between Amazon GuardDuty and Amazon Inspector?**
*   A. GuardDuty is for assessing compliance, while Inspector is for threat detection.
*   B. GuardDuty is a threat detection service, while Inspector assesses applications for vulnerabilities.
*   C. GuardDuty requires agents on EC2 instances, while Inspector is agentless.
*   D. GuardDuty is for network security, while Inspector is for data encryption.

**Answer: B.** Amazon GuardDuty is an intelligent threat detection service that analyzes AWS CloudTrail logs, VPC Flow Logs, and DNS logs. Amazon Inspector assesses applications on EC2 instances for vulnerabilities or deviations from best practices, often using an agent.

**4. When is it necessary to get prior approval from AWS?**
*   A. When enabling AWS Shield Advanced.
*   B. When conducting penetration testing on your own EC2 instances.
*   C. When creating an IAM role with broad permissions.
*   D. When configuring AWS WAF rules.

**Answer: B.** AWS requires customers to submit a request for prior approval before conducting penetration tests on their AWS infrastructure. This is to prevent triggering automated security defenses.

**5. A Lambda function is triggered when an object is uploaded to an S3 bucket. It needs to write metadata to a DynamoDB table. How should the Lambda function be granted access to the table?**
*   A. Create an IAM user with permissions to write to the DynamoDB table and store the access key in the Lambda environment variables.
*   B. Create a resource policy on the DynamoDB table that grants write permissions to the Lambda function's ARN.
*   C. Create a VPC endpoint for DynamoDB and configure the Lambda function to use the VPC.
*   D. Create an IAM execution role with permissions to write to the DynamoDB table and associate it with the Lambda function.

**Answer: D.** The recommended and secure practice is to assign an IAM execution role to the Lambda function. This role should have the necessary permissions policies attached. Options involving long-term access keys (A) are insecure, and resource policies (B) are not the standard method for Lambda-to-DynamoDB access.

#### ⚙️ Practical & Applied Security (CLI, Configurations, Code)

**6. You are given the following AWS CLI command. What security feature is being enabled?**
    `aws dynamodb create-table --table-name Customer --attribute-definitions AttributeName=CustomerID,AttributeType=S --key-schema AttributeName=CustomerID,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 --sse-specification Enabled=true`
*   A. Data encryption in transit for the Customer table.
*   B. Data encryption at rest for the Customer table.
*   C. Security groups for the Customer table.
*   D. IAM authentication for the Customer table.

**Answer: B.** The `--sse-specification Enabled=true` parameter enables server-side encryption (SSE) at rest for the DynamoDB table. By default, this uses AWS-owned keys, but can be configured to use AWS KMS.

**7. To monitor all API activity across all current and FUTURE AWS regions, what is the best practice for configuring AWS CloudTrail?**
*   A. Create a single CloudTrail trail and configure it to apply to all regions.
*   B. Manually create a CloudTrail trail in each existing region and use CloudFormation for future regions.
*   C. Use AWS Config to enable CloudTrail in all regions.
*   D. Rely on the default CloudTrail event history that is automatically enabled.

**Answer: A.** When you create a trail, you can apply it to all AWS regions. This trail will automatically log events from all regions and will extend to any new region that becomes available without any additional action required.

**8. A Security Engineer needs to design a solution for the Incident Response team to audit changes to a user's IAM permissions. How can this be accomplished?**
*   A. Run `GenerateCredentialReport` via the AWS CLI daily and store the output in Amazon S3.
*   B. Use AWS Config to review the IAM policies assigned to users before and after an incident.
*   C. Use Amazon EC2 Systems Manager to deploy images and review CloudTrail logs for changes.
*   D. Copy AWS CloudFormation templates to S3 and audit for changes.

**Answer: B.** AWS Config can record configuration changes to IAM resources, such as users, groups, roles, and policies. It provides a history of changes, allowing auditors to see what permissions a user had at any point in time.

**9. Which AWS Systems Manager capability should be used as a security best practice to connect to an EC2 instance for administrative tasks instead of using a bastion host?**
*   A. AWS Systems Manager Patch Manager.
*   B. AWS Systems Manager Run Command.
*   C. AWS Systems Manager Session Manager.
*   D. AWS Systems Manager Inventory.

**Answer: C.** Session Manager provides secure and auditable instance management without the need to open inbound SSH ports or manage bastion hosts and SSH keys. Connections are made through the Systems Manager service.

**10. What is a key consideration for using AWS KMS key policies?**
*   A. Key policies are only used for IAM users, not IAM roles.
*   B. A key policy is mandatory for every customer master key (CMK) and defines who has access to the key.
*   C. Key policies are less important than IAM policies.
*   D. Key policies are automatically managed by AWS and cannot be edited.

**Answer: B.** The key policy is the primary way to control access to a KMS CMK. It is required, and even if IAM policies grant permissions, the key policy must also allow access for the request to be successful.

**11. What is the primary purpose of an IAM Role's "trust policy"?**
*   A. To define which AWS services or accounts can assume the role.
*   B. To specify the permissions that the role has on AWS resources.
*   C. To set up multi-factor authentication (MFA) for the role.
*   D. To define the password policy for IAM users who can assume the role.

**Answer: A.** The trust policy (or trust relationship) is a resource-based policy attached to the IAM Role that defines which principals (users, services, accounts) are allowed to assume this role. The permissions policy defines what the role can do once assumed.

**12. A company wants to ensure that all S3 buckets are encrypted by default. What is the most effective way to enforce this?**
*   A. Use AWS Config to monitor and alert on unencrypted buckets.
*   B. Implement an S3 Bucket Policy that denies `PutObject` requests without the `x-amz-server-side-encryption` header.
*   C. Use AWS Organizations SCPs to block the creation of unencrypted S3 buckets.
*   D. Create an IAM policy that denies the `s3:CreateBucket` permission unless the user specifies encryption.

**Answer: B.** While SCPs can restrict some actions, they cannot enforce specific parameters like encryption headers for S3. A bucket policy with an explicit deny condition on `PutObject` without the encryption header is a strong, direct enforcement mechanism. AWS Config (A) is for monitoring, not prevention.

**13. Which AWS service is designed to protect web applications from common exploits like SQL Injection and Cross-Site Scripting (XSS)?**
*   A. AWS Shield
*   B. AWS WAF (Web Application Firewall)
*   C. Amazon GuardDuty
*   D. AWS Network Firewall

**Answer: B.** AWS WAF allows you to create rules to filter malicious web traffic targeting your applications. AWS Shield protects against DDoS attacks, GuardDuty is for threat detection, and Network Firewall is a more general stateful firewall.

**14. What is a key difference between AWS Secrets Manager and AWS Systems Manager Parameter Store?**
*   A. Only Secrets Manager can store sensitive data.
*   B. Secrets Manager has built-in rotation capabilities for secrets (e.g., RDS passwords), while Parameter Store does not.
*   C. Parameter Store can only store plaintext strings, not encrypted values.
*   D. Only Parameter Store can be used with Lambda functions.

**Answer: B.** The primary differentiator is the automated secret rotation feature in Secrets Manager. Both services can store secrets encrypted with KMS. Parameter Store is excellent for configuration data and secrets but requires custom logic for rotation.

**15. For regulatory compliance, you need to ensure that an EC2 instance in a VPC cannot be accessed from the public internet. How can this be achieved?**
*   A. Place the instance in a private subnet (with no route to an Internet Gateway).
*   B. Assign a private IP address to the instance.
*   C. Configure the instance's security group to deny all inbound traffic from 0.0.0.0/0.
*   D. Use AWS Shield to block public access.

**Answer: A.** The most fundamental network-level control is the subnet's route table. A private subnet has no route to an Internet Gateway (IGW), making the instance unreachable from the internet regardless of security group settings. Security groups (C) provide an additional layer but are not the primary network isolation control.

### ⚙️ Practical & Applied Security (CLI, Configurations, Code) - Продолжение

**16. You need to verify the cryptographic fingerprint of an SSH key pair you just created using the AWS CLI. Which command is correct?**
*   A. `aws ec2 describe-key-pairs --key-name MyKeyPair`
*   B. `aws ec2 get-console-output --instance-id i-1234567890abcdef0`
*   C. `aws ec2 describe-instances --filters "Name=key-name,Values=MyKeyPair"`
*   D. `openssl rsa -in MyKeyPair.pem -pubout -outform DER | openssl md5 -c` (This is a local OpenSSL command, not AWS CLI)

**Answer: A.** The `describe-key-pairs` CLI command returns the key's name, fingerprint, and other details. Option D is a valid way to calculate a fingerprint locally, but it's not an AWS CLI command.

**17. You are analyzing a VPC Flow Log record. What does the "ACCEPT" action in the log entry signify?**
*   A. The traffic was allowed by a Network ACL (NACL).
*   B. The traffic was allowed by a Security Group.
*   C. The traffic was recorded by CloudTrail.
*   D. The traffic was permitted by either a Security Group or a Network ACL at the specific point of evaluation.

**Answer: D.** VPC Flow Logs record traffic at the network interface level. An "ACCEPT" means the traffic was permitted by the stateful security group rules or the stateless NACL rules for that particular network interface. The log doesn't distinguish which one allowed it.

**18. Which command would you use to force an immediate rotation of a secret in AWS Secrets Manager via the CLI?**
*   A. `aws secretsmanager update-secret --secret-id MySecret --rotation-enabled`
*   B. `aws secretsmanager rotate-secret --secret-id MySecret`
*   C. `aws secretsmanager get-secret-value --secret-id MySecret`
*   D. `aws secretsmanager put-resource-policy --secret-id MySecret`

**Answer: B.** The `rotate-secret` command immediately rotates a secret. `update-secret` with `--rotation-enabled` is used to initially turn on rotation, which then happens automatically based on a schedule.

**19. You need to ensure that all data uploaded to an S3 bucket is encrypted with a specific AWS KMS Customer Master Key (CMK). Which parameter would you use in the AWS CLI `put-bucket-encryption` command?**
*   A. `--server-side-encryption AWS256`
*   B. `--sse-specification '{"SSEAlgorithm": "aws:kms", "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789012:key/abcd1234-..."}'`
*   C. `--encryption-type KMS-CUSTOMER-KEY`
*   D. `--kms-key-id arn:aws:kms:us-east-1:123456789012:key/abcd1234-...`

**Answer: B.** The correct syntax for the `put-bucket-encryption` command includes the `--sse-specification` parameter with a JSON structure specifying the algorithm and the specific KMS key ARN.

**20. What is the primary function of the `aws kms generate-data-key` CLI command?**
*   A. To create a new Customer Master Key (CMK) in KMS.
*   B. To generate a data key that can be used to encrypt data locally, outside of AWS.
*   C. To encrypt a large file directly using KMS.
*   D. To rotate a CMK.

**Answer: B.** `generate-data-key` returns a plaintext data key and an encrypted copy of that data key. You use the plaintext key to encrypt data locally (e.g., with OpenSSL) and then store the encrypted data key alongside the ciphertext. This is more efficient than sending large data to KMS directly.

---

### 🚨 Advanced Scenarios & Hybrid Architecture

**21. A company uses AWS Organizations and wants to prevent member accounts from disabling AWS CloudTrail logging. What is the most effective method?**
*   A. Use AWS Config rules in the management account.
*   B. Create a Service Control Policy (SCP) that denies the `cloudtrail:StopLogging` and `cloudtrail:DeleteTrail` actions for all member accounts.
*   C. Use IAM policies in each member account.
*   D. Enable CloudTrail Lake in the management account.

**Answer: B.** SCPs are the primary tool in AWS Organizations to set guardrails and enforce permissions boundaries across member accounts. They can deny high-level actions like stopping or deleting trails.

**22. In a hybrid architecture, what is the recommended way to securely connect an on-premises Active Directory to AWS for federated user access?**
*   A. Use AWS IAM Identity Center (successor to AWS Single Sign-On).
*   B. Set up a site-to-site VPN and use AD Connector.
*   C. Use AWS Managed Microsoft AD and establish a trust relationship with the on-premises AD.
*   D. Replicate all users to IAM using a custom script.

**Answer: C.** AWS Managed Microsoft AD is a standalone Active Directory in the AWS cloud. You can establish a forest trust relationship with your on-premises AD, allowing seamless federation and a more robust setup than AD Connector for complex scenarios. AWS IAM Identity Center can then be used for SSO.

**23. What does the "Principal of Least Privilege" mean in the context of IAM?**
*   A. Granting users permissions to all services except IAM.
*   B. Granting only the permissions necessary to perform a specific task.
*   C. Using IAM roles instead of IAM users.
*   D. Enabling MFA for all root and IAM users.

**Answer: B.** This is a fundamental security principle. Users, groups, and roles should be granted only the minimum permissions they need to do their job and nothing more.

**24. You need to investigate a potential security incident and want to see all API calls made by a specific IAM user within a specific time frame. Which AWS service is best suited for this?**
*   A. Amazon CloudWatch Logs
*   B. AWS X-Ray
*   C. AWS CloudTrail
*   D. AWS Trusted Advisor

**Answer: C.** AWS CloudTrail is the service that records API activity and account events. You can filter the event history or search CloudTrail logs in an S3 bucket by user, time, and API name.

**25. Which of the following is a key advantage of using AWS Systems Manager Session Manager over a traditional bastion host for EC2 instance access?**
*   A. It provides faster network throughput.
*   B. It does not require inbound security group rules to be opened.
*   C. It allows for graphical desktop (RDP) connections.
*   D. It is less expensive than a small EC2 instance.

**Answer: B.** Session Manager connects to instances through the Systems Manager service, eliminating the need to expose the instance with an inbound SSH/RDP port in the security group. This significantly reduces the attack surface.

---

### 🔍 Logging, Monitoring & Incident Response

**26. Which AWS service can be used to create a centralized log archive from multiple accounts and perform complex queries using a SQL-like language?**
*   A. Amazon CloudWatch Logs Insights
*   B. AWS CloudTrail Lake
*   C. Amazon Athena
*   D. Amazon OpenSearch Service

**Answer: B.** AWS CloudTrail Lake is a specifically managed service for ingesting, storing, and querying CloudTrail events from multiple accounts and regions. While Athena can query S3 logs, CloudTrail Lake is purpose-built and pre-optimized for this task.

**27. You want to be alerted via Amazon SNS whenever an IAM policy is modified in your AWS account. What is the best way to set this up?**
*   A. Create a CloudWatch Event (Amazon EventBridge) rule that triggers on the `iam:PutPolicy` API event and sends a notification to an SNS topic.
*   B. Configure Amazon GuardDuty to send findings to SNS.
*   C. Use AWS Config to detect changes to IAM policies and send an SNS notification.
*   D. Write a Lambda function that polls the IAM API every minute.

**Answer: C.** AWS Config is designed for tracking configuration changes, including IAM policies. It can be configured to send notifications via SNS when a resource violates a rule (e.g., a policy has been changed). This is more robust than a simple EventBridge rule for configuration drift.

**28. What is the purpose of a "dead-letter queue" (DLQ) in an AWS security context?**
*   A. A queue for storing security alerts that have been resolved.
*   B. A queue that holds messages or events that could not be processed successfully by a service like AWS Lambda or Amazon SQS.
*   C. A special S3 bucket for storing corrupted files.
*   D. A KMS key that is no longer in use.

**Answer: B.** In event-driven architectures (e.g., Lambda triggered by SQS), a DLQ is a crucial resilience feature. If a message fails processing repeatedly, it's moved to the DLQ for isolated analysis, preventing a backlog and allowing security teams to investigate problematic events.

**29. To comply with GDPR's "Right to Erasure," you need to delete a specific customer's data from an Amazon S3 bucket. However, the bucket uses S3 Versioning. What must you do to permanently delete the object?**
*   A. Use the `s3:DeleteObject` permission. Versioning will handle the rest.
*   B. Delete the object and then empty the bucket's recycle bin.
*   C. Permanently delete the object and all its versions by specifying the version ID(s).
*   D. Suspend versioning and then delete the object.

**Answer: C.** With versioning enabled, a simple delete operation only adds a delete marker. To permanently erase the data, you must delete all versions of the object. This often requires using the version ID with the delete command.

**30. You suspect an IAM user's access keys have been compromised. What is the IMMEDIATE first step you should take?**
*   A. Delete the IAM user.
*   B. Deactivate the compromised access keys for the IAM user.
*   C. Modify the IAM user's permissions policy.
*   D. Enable MFA for the IAM user.

**Answer: B.** The immediate action to stop any ongoing unauthorized access is to deactivate or delete the compromised keys. Deleting the user (A) might be an overreaction and could disrupt legitimate workflows. Changing policies (C) or enabling MFA (D) are important follow-up steps but do not immediately revoke the existing key-based access.

---

### 💡 Conclusion & Preparation Advice

This list of questions, created by **Ivan Piskunov (c) 2021**, is an approximation of the topics covered in the SCS-C01 exam during the 2020-2021 period. While the exam is challenging and requires deep, practical knowledge, it is highly relevant and achievable with dedicated preparation.

The key to success is a **"security-first" mindset**—always choosing the most secure option unless explicitly told otherwise for cost or simplicity. Hands-on practice with core services like IAM, KMS, CloudTrail, and Systems Manager is crucial.

#### Recommended Resources (Circa 2020-2021):
*   **Official Exam Guide & Sample Questions:** [AWS Certified Security - Specialty](https://aws.amazon.com/certification/certified-security-specialty/) .
*   **AWS Training & Certification Blog:** [10 tips to study for the AWS Certified Security – Specialty Certification](https://aws.amazon.com/blogs/training-and-certification/10-tips-to-study-for-the-aws-certified-security-specialty-certification/) .
*   **AWS Whitepapers:** *AWS Security Best Practices* and *AWS Well-Architected Framework*.
*   **Practice Exams:** Tutorials Dojo (Jon Bonso) and Whizlabs were highly recommended by the community for their realistic questions and detailed explanations.
*   **Book:** *AWS Certified Security – Specialty Exam Guide* by Stuart Scott.
