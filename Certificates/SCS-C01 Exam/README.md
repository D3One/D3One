
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
