---

# 🔐 Secure AWS VPC Infrastructure with Cloud Security, Terraform, and AI-Driven Insights

## 🚀 Overview

This project demonstrates a **secure, scalable, and production-ready AWS VPC environment**, deployed first **manually** and then automated using **Terraform**. It focuses on **core cloud security principles**, including **network segmentation, IAM security, traffic flow control, and compliance checks**. The project integrates **Prowler for AWS security auditing** and **AI-based security insights** to detect and suggest remediations for potential misconfigurations.

---

## 📚 Table of Contents

- [Project Architecture](#architecture)
- [Key Objectives](#key-objectives)
- [Step-by-Step Deployment](#step-by-step-deployment)
- [Security Tools Used](#security-tools-used)
- [AI Security Integration](#ai-security-integration)
- [Further Enhancements](#further-enhancements)
- [Learning Outcomes](#learning-outcomes)
- [Prerequisites](#prerequisites)
- [Credits & References](#credits--references)

---

## 🏗 Architecture

```
User (Admin) ─── SSH ───> Bastion Host (Public Subnet)
                             │
                             │ SSH/HTTP
                             ▼
                      Web Servers (Private Subnet)
                             │
                          NAT Gateway (Public Subnet)
                             │
                       Internet Gateway (VPC)
```

### Components:

| Component                    | Type                   | Description                               |
|-----------------------------|------------------------|-------------------------------------------|
| VPC                         | 10.0.0.0/16            | Main virtual network                      |
| Public Subnets              | 10.0.1.0/24, 10.0.2.0/24| For Bastion and NAT Gateway               |
| Private Subnets             | 10.0.3.0/24, 10.0.4.0/24| For internal EC2 Web Servers              |
| Internet Gateway            | -                      | Internet access for public subnet         |
| NAT Gateway                 | -                      | Outbound internet access for private subnet|
| EC2 Bastion Host            | t2.micro               | Secure admin access point                 |
| EC2 Web Servers             | t2.micro               | Hosting internal application (e.g., nginx)|
| IAM Roles & S3              | -                      | For secure credential handling            |
| Security Groups             | -                      | Layered firewall for EC2 and subnets       |

---

## 🎯 Key Objectives

- ✅ Design and deploy **secure VPC architecture**
- ✅ Implement **public and private subnet segregation**
- ✅ Secure inbound and outbound access using **NACLs and Security Groups**
- ✅ **IAM roles** for EC2 instances with least privilege
- ✅ **Audit and analyze** AWS environment using **Prowler**
- ✅ Integrate **AI-driven security analysis** to detect misconfigurations
- ✅ Infrastructure as Code (IaC) using **Terraform** for automation

---

## 🛠 Step-by-Step Deployment

### **Phase 1: Manual AWS Deployment**

1. **Create VPC and Subnets**
   - Public & Private Subnets (Multi-AZ)
2. **Deploy Internet and NAT Gateways**
3. **Configure Route Tables for traffic flow**
4. **Set up Bastion Host in Public Subnet**
5. **Launch Web Servers in Private Subnets**
6. **Apply Security Groups & IAM Roles**
7. **Store EC2 logs in S3 with lifecycle policy**

---


## Challenges Faced during phase 1:

## SSH Connection Troubleshooting Summary

1. **Bastion Host Connection Failure**: Initial SSH connection to the bastion host (13.201.119.178) failed with "Connection timed out" error.

2. **Security Group Restriction**: Security group configuration was allowing SSH access only from a specific IP address (111.92.13.48/32), not from your CloudShell IP.

3. **Private Server Access**: Attempted direct connection to private web server (10.0.3.34) from CloudShell, which is not possible as private IPs are not accessible from the internet.

4. **Missing SSH Key**: When connecting from the bastion host to the private web server, the SSH key wasn't available on the bastion host.

## Solutions Implemented

1. **Security Group Update**: Modified the security group to allow SSH access from your current IP address.

Update the security group to allow your current IP:
```json
aws ec2 authorize-security-group-ingress \
    --group-id sg-074d14580bded3ade \
    --protocol tcp \
    --port 22 \
    --cidr $(curl -s https://checkip.amazonaws.com)/32
```
This adds your current IP to the allowed list while keeping the existing rule.

2. **Two-Step SSH Connection**: Established connection to the bastion host first, then from there to the private web server.

3. **SSH Agent Forwarding**: Used SSH agent forwarding to authenticate to the private server without copying the private key to the bastion host.
```json
eval $(ssh-agent) 
ssh-add keypair.pem 
ssh -A ec2-user@13.201.119.178

```
This approach followed the proper security pattern for accessing private resources in AWS, using a bastion host as the secure entry point and keeping sensitive credentials (SSH keys) protected.

---

## Challenges Faced in AWS Config Setup and Resolutions

## Challenge 1: Configuration Recorder Creation Failed 
### **Issue:**  
When attempting to set up AWS Config, the following error was encountered:  

```
Insufficient delivery policy to s3 bucket: cloudtrail-logs-jib, unable to write to bucket, provided s3 key prefix is 'null', provided kms key is 'null'.
```

This occurred because AWS Config did not have the necessary permissions to write logs to the specified **S3 bucket**.

### **Resolution:**  
Updated the **S3 bucket policy** to grant AWS Config the required permissions while keeping existing CloudTrail permissions intact.  

#### **Updated S3 Bucket Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSCloudTrailAclCheck",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudtrail.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::cloudtrail-logs-jib",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudtrail:ap-south-1:816069160759:trail/SecurityTrail"
                }
            }
        },
        {
            "Sid": "AWSCloudTrailWrite",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudtrail.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::cloudtrail-logs-jib/AWSLogs/816069160759/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control",
                    "AWS:SourceArn": "arn:aws:cloudtrail:ap-south-1:816069160759:trail/SecurityTrail"
                }
            }
        },
        {
            "Sid": "AWSConfigBucketPermissionsCheck",
            "Effect": "Allow",
            "Principal": {
                "Service": "config.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::cloudtrail-logs-jib"
        },
        {
            "Sid": "AWSConfigBucketDelivery",
            "Effect": "Allow",
            "Principal": {
                "Service": "config.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::cloudtrail-logs-jib/AWSLogs/816069160759/Config/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        }
    ]
}
```
### **Outcome:**  
- AWS Config now has **write access** to the S3 bucket, resolving the configuration recorder failure.  
- The issue was fixed without impacting CloudTrail logging.  

---

## Challenge 2: IAM Role for AWS Config Not Assigned  
### **Issue:**  
When setting up AWS Config, an error was encountered:  

```
An unexpected error occurred and the role was not created (or updated). Try again or contact AWS support if the error persists.
```

This happened because AWS Config was unable to automatically create or assign the **service-linked IAM role** required for its operation.

### **Resolution:**  
#### **Step 1: Manually Create the IAM Role**
1. Navigate to **AWS IAM Console → Roles**.  
2. Click **Create Role** → Select **AWS Service** → Choose **Config**.  
3. Click **Next** → Attach the following policies:
   - **AWSConfigRole**
   - **AmazonS3FullAccess** (temporary, only for testing)  
4. Click **Next**, name the role **AWSServiceRoleForConfig**, and create it.  

#### **Step 2: Update the Trust Relationship**
1. Go to **IAM Console** → **Roles** → Search for `AWSServiceRoleForConfig`.  
2. Click on **Trust relationships** → Click **Edit trust policy**.  
3. Ensure the trust policy contains:  
   ```json
   {
      "Version": "2012-10-17",
      "Statement": [
         {
            "Effect": "Allow",
            "Principal": {
               "Service": "config.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
         }
      ]
   }
   ```
4. Click **Update trust policy**.  

#### **Step 3: Retry AWS Config Setup**
1. Go to **AWS Config Console**.  
2. Click **Set up AWS Config**.  
3. Select **Use an existing IAM role** → Choose **AWSServiceRoleForConfig**.  
4. Proceed with the setup.  

### **Outcome:**  
- AWS Config is now able to assume the correct IAM role.  
- The service successfully initializes without permission errors.  

---

## **Final Takeaways**
- **Ensure AWS Config has the correct S3 bucket permissions** to avoid log delivery failures.  
- **Manually create and assign the AWS Config IAM role** if automatic role creation fails.  
- **Verify trust policies and permissions** when dealing with AWS service roles.  

These steps successfully resolved both challenges, enabling AWS Config to function properly. ✅


### **Phase 2: Security Auditing with Prowler**

- **Run Prowler** for:
  - IAM Misconfigurations
  - Network Exposure
  - Encryption and Compliance Violations (CIS, PCI, GDPR)
- **Generate Security Report**

---

# Prowler Installation and Configuration Troubleshooting

## Summary of Challenges and Solutions

I encountered several challenges while setting up and running Prowler, a security assessment tool for AWS environments. This document outlines the issues faced and how they were resolved.

## Dependency Issues

### Missing Python Modules
When attempting to run Prowler using the command `python3 prowler-cli.py -M csv,json,html -o prowler-report`, we encountered a series of missing dependency errors:

1. **detect_secrets module**
   - Error: `ModuleNotFoundError: No module named 'detect_secrets'`
   - Solution: Installed the missing module using `pip3 install detect-secrets`

2. **slack_sdk module**
   - Error: `ModuleNotFoundError: No module named 'slack_sdk'`
   - Solution: Installed the missing module using `pip3 install slack_sdk`

3. **kubernetes client module**
   - Error: `ImportError: cannot import name 'client' from 'kubernetes'`
   - Solution: Reinstalled the kubernetes module using:
     ```bash
     pip3 uninstall -y kubernetes
     pip3 install kubernetes
     ```

### Requirements File Not Found
We attempted to install all dependencies at once using the requirements file:
```bash
pip3 install -r requirements.txt
```

However, this failed with the error: `ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'`

## Command Format Issues

After resolving the dependency issues, we encountered a formatting issue with the command:

- Error: `prowler [-h] [--version] {aws,azure,gcp,kubernetes,microsoft365,dashboard} ... aws: error: argument --output-formats/--output-modes/-M: invalid choice: 'csv,json,html'`

- Solution: Modified how the output formats were specified by separating them with spaces instead of commas:
  ```bash
  python3 prowler-cli.py -M csv json-ocsf html -o prowler-report
  python3 prowler-cli.py --region ap-south-1 -M csv json-ocsf html -o prowler-report
  ```

## Pydantic Schema Generation Error

After getting past the initial dependency and command format issues, we encountered a Pydantic schema generation error with Prowler 5.5.0:

### Possible Causes:
1. **Pydantic Version Mismatch**: Prowler was using Pydantic v1 syntax, while our system had Pydantic v2 installed
2. **Code Compatibility Issue**: The Prowler code wasn't allowing arbitrary types in Pydantic models
3. **Environment Issues**: Dependencies may not have been correctly installed via Poetry

### Solutions Implemented:
1. **Downgraded Pydantic** to version 1:
   ```bash
   pip install "pydantic<2"
   ```

2. **Modified Prowler Code** (when applicable) to allow arbitrary types in `compliance_models.py`:
   ```python
   from pydantic import BaseModel, ConfigDict

   class CIS_Requirement_Attribute_Profile(BaseModel):
       model_config = ConfigDict(arbitrary_types_allowed=True)
   ```

3. **Used Poetry** for proper dependency management:
   ```bash
   poetry install
   poetry run python3 prowler-cli.py -M csv json-ocsf html -o prowler-report
   ```

<img width="586" alt="image" src="https://github.com/user-attachments/assets/cb6b5a98-1a09-4635-a036-604fde1051f8" />


## Conclusion

The main issues encountered were related to missing dependencies and the compatibility between Prowler's code and Pydantic versions. By systematically addressing each dependency issue and correcting the command format, we were able to successfully run Prowler for security assessments. The Pydantic version mismatch was the most significant challenge, requiring either a version downgrade or code modification to resolve.

<img width="692" alt="image" src="https://github.com/user-attachments/assets/92146506-c9f9-46f3-8f40-8f996ce3d6be" />

### **Phase 3: AI-Driven Security Insights**

- **Analyze Prowler output** with AI tool (custom script/ChatGPT/LLM API)
- Generate:
  - Top 10 security risks
  - AI-suggested remediation steps
  - Risk categorization: Critical, High, Medium, Low

---
 ```json

### Top 10 Security Risks Identified in Prowler Scan:

1. **IAM Configuration Issues**  
   - **Critical**: 0
   - **High**: 14
   - **Medium**: 12
   - **Low**: 6  
   **Risk**: Misconfigured IAM policies or roles can lead to excessive permissions, potentially allowing unauthorized access to sensitive resources. 

2. **CloudWatch Configuration Issues**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 17  
   - **Low**: 0  
   **Risk**: Lack of proper monitoring and logging configuration may lead to undetected security events.

3. **S3 Bucket Misconfiguration**  
   - **Critical**: 0  
   - **High**: 1  
   - **Medium**: 22  
   - **Low**: 12  
   **Risk**: Misconfigured S3 buckets can expose sensitive data if the permissions are too permissive or incorrect.

4. **EC2 Instance Misconfigurations**  
   - **Critical**: 0  
   - **High**: 1  
   - **Medium**: 12  
   - **Low**: 4  
   **Risk**: EC2 instances may be running with outdated or insecure configurations, making them vulnerable to attacks.

5. **CloudTrail Configuration Issues**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 3  
   - **Low**: 1  
   **Risk**: Insufficient logging and monitoring of AWS CloudTrail could lead to a failure to detect malicious activity.

6. **IAM Policies Not Following Best Practices**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 1  
   - **Low**: 0  
   **Risk**: IAM policies not aligned with security best practices can lead to unauthorized access to AWS resources.

7. **Network Firewall Misconfiguration**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 1  
   - **Low**: 0  
   **Risk**: Misconfigured network firewalls may allow unauthorized inbound or outbound traffic.

8. **Backup Configuration Issues**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 0  
   - **Low**: 1  
   **Risk**: Inadequate backup configurations or missing backups may result in data loss in case of a breach.

9. **GuardDuty Detection Misconfiguration**  
   - **Critical**: 0  
   - **High**: 0  
   - **Medium**: 1  
   - **Low**: 0  
   **Risk**: Misconfigured GuardDuty detection might miss detecting potential threats to the environment.

10. **VPC Misconfigurations**  
    - **Critical**: 0  
    - **High**: 0  
    - **Medium**: 1  
    - **Low**: 0  
    **Risk**: Insecure VPC configurations can expose resources to the internet or malicious users.

---

### AI-Suggested Remediation Steps:

1. **IAM Configuration Issues**:
   - Regularly review and refine IAM roles and permissions based on the principle of least privilege.
   - Implement identity federation where necessary to limit direct IAM account usage.
   - Use IAM Access Analyzer to continuously audit permissions and roles.

2. **CloudWatch Configuration Issues**:
   - Ensure that CloudWatch is properly configured to capture and store logs for all resources.
   - Set up CloudWatch Alarms to notify of suspicious activities or abnormal behavior.
   - Enable CloudWatch Logs Insights to monitor logs in real-time for security events.

3. **S3 Bucket Misconfiguration**:
   - Ensure S3 buckets are private by default and that ACLs are not set to public.
   - Enable S3 Bucket Logging to track access and modifications to the buckets.
   - Implement AWS S3 Block Public Access settings across all buckets.

4. **EC2 Instance Misconfigurations**:
   - Ensure EC2 instances are configured with security groups and NACLs that restrict inbound and outbound traffic.
   - Apply security patches and updates regularly.
   - Use Amazon Inspector for automated vulnerability scanning of EC2 instances.

5. **CloudTrail Configuration Issues**:
   - Ensure that CloudTrail is enabled in all regions, and logs are delivered to a secure S3 bucket.
   - Set up alerts for unauthorized API calls using CloudWatch Logs.
   - Regularly monitor CloudTrail logs for any suspicious activity.

6. **IAM Policies Not Following Best Practices**:
   - Review IAM policies for overly permissive statements and implement stricter controls.
   - Use AWS managed policies where possible to simplify access control.
   - Use permissions boundary to limit the scope of permissions.

7. **Network Firewall Misconfiguration**:
   - Regularly audit security group and NACL configurations to ensure that only trusted IPs have access to critical resources.
   - Use AWS WAF to protect web-facing resources from common threats.
   - Enable VPC Flow Logs for network traffic monitoring.

8. **Backup Configuration Issues**:
   - Automate backups for critical data using AWS Backup.
   - Regularly test backup restoration to ensure the integrity and reliability of backup systems.
   - Set backup retention policies to avoid over-retention of outdated data.

9. **GuardDuty Detection Misconfiguration**:
   - Ensure GuardDuty is enabled in all regions and configured to send findings to CloudWatch for alerting.
   - Regularly review GuardDuty findings and take appropriate remediation actions.
   - Integrate GuardDuty with other security tools like AWS Security Hub for a consolidated view.

10. **VPC Misconfigurations**:
    - Use VPC flow logs to track network traffic and monitor for unusual behavior.
    - Restrict VPC traffic using security groups, NACLs, and VPC Peering.
    - Use AWS Transit Gateway for centralized network management and security.

---

### Risk Categorization:

- **Critical**: None identified.
- **High**: IAM (14), EC2 (1), GuardDuty (1)
- **Medium**: CloudWatch (17), IAM (12), S3 (22), EC2 (12), CloudTrail (3), Bedrock (1), VPC (1)
- **Low**: S3 (12), Backup (1), Config (1), Macie (1), Support (1), VPC (1), ResourceExplorer2 (1), SSM (1), Network-Firewall (1), VPC (1), Backup (1)

---

**Note**: These findings should be prioritized based on your organization's risk tolerance, sensitivity of resources, and compliance requirements. The AI-suggested remediation steps aim to address the most critical issues in a systematic manner to improve overall security posture.
 ```
### **Phase 4: Terraform Automation**

- Rebuild entire infrastructure using Terraform:
  - VPC, Subnets, Route Tables, IGW, NAT
  - EC2 instances with provisioning
  - IAM roles, S3, and Security Groups
- Outputs:
  - Terraform `.tf` files and `state` management

---

## 🔐 Security Tools Used

| Tool         | Purpose                                      |
|--------------|----------------------------------------------|
| **AWS IAM**  | Least privilege roles & policies              |
| **Prowler**  | AWS security auditing & compliance reporting |
| **AWS S3**   | Centralized log storage                      |
| **Security Groups & NACLs** | Traffic control for instances and subnets |

---

## 🤖 AI Security Integration

After running **Prowler**, AI tools (e.g., ChatGPT, Amazon Bedrock, LangChain) analyze reports to:
- Summarize vulnerabilities
- Provide prioritized action plan
- Suggest IAM, VPC, and resource-specific improvements
- Generate human-readable security posture report

---

## 🚀 Further Enhancements

| Feature                               | Description                                   |
|--------------------------------------|-----------------------------------------------|
| AWS Config                           | Resource compliance monitoring                 |
| AWS GuardDuty                        | Threat detection and security monitoring     |
| AWS Security Hub                     | Centralized security alerts & posture         |
| SSM Session Manager                  | SSH-less access to EC2                        |
| CI/CD with GitHub Actions/Terraform Cloud | Infra as Code pipeline                     |
| AWS WAF + Shield                     | Protect web workloads                        |
| Centralized CloudWatch Monitoring    | Logs, metrics, and alerts                    |

---

## 🎓 Learning Outcomes

| Area                                | Skills Gained                                  |
|-------------------------------------|------------------------------------------------|
| **Cloud Networking**                 | VPC, Subnets, NAT, Route Tables                |
| **Cloud Security**                   | IAM, Security Groups, NACLs, Encryption        |
| **Automation**                       | Terraform, Modular Design, IaC best practices |
| **Cloud Compliance**                 | CIS, PCI, GDPR checks with Prowler             |
| **AI Security**                      | AI-driven risk analysis and remediation       |
| **Cloud Storage**                    | S3 lifecycle and secure storage               |
| **Infrastructure Hardening**         | Multi-layer defense-in-depth                  |

---

## 🔑 Prerequisites

- **AWS Free Tier Account** (Eligible services used only)
- **AWS CLI and IAM Admin User**
- **Terraform (latest version)**
- **Prowler CLI installed**
- Optional: **LangChain/LLM/AI APIs for AI security analysis**

---

## 📚 Credits & References

- [AWS VPC Best Practices](https://aws.amazon.com/answers/networking/vpc-best-practices/)
- [Prowler AWS Security Tool](https://github.com/prowler-cloud/prowler)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS Security Hub](https://aws.amazon.com/security-hub/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

## 🏆 Final Deliverables

| Deliverable                                | Description                             |
|--------------------------------------------|-----------------------------------------|
| **Manual Setup Screenshots/Documentation** | Evidence and step breakdown             |
| **Terraform Code**                         | Fully codified, reusable infrastructure |
| **Prowler Security Report**                | Detailed audit of AWS environment       |
| **AI-generated Remediation Report**        | AI-processed security analysis          |
| **Architecture Diagram**                   | Visual depiction of setup               |

---

## 📩 Contact & Feedback

Feel free to fork this repo and enhance further. Contributions welcome!

📧 **Contact**: [Your Email or LinkedIn]

---
