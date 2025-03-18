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

aws ec2 authorize-security-group-ingress \
    --group-id sg-074d14580bded3ade \
    --protocol tcp \
    --port 22 \
    --cidr $(curl -s https://checkip.amazonaws.com)/32

This adds your current IP to the allowed list while keeping the existing rule.

2. **Two-Step SSH Connection**: Established connection to the bastion host first, then from there to the private web server.

3. **SSH Agent Forwarding**: Used SSH agent forwarding to authenticate to the private server without copying the private key to the bastion host.

eval $(ssh-agent) 
ssh-add keypair.pem 
ssh -A ec2-user@13.201.119.178


This approach followed the proper security pattern for accessing private resources in AWS, using a bastion host as the secure entry point and keeping sensitive credentials (SSH keys) protected.


### **Phase 2: Security Auditing with Prowler**

- **Run Prowler** for:
  - IAM Misconfigurations
  - Network Exposure
  - Encryption and Compliance Violations (CIS, PCI, GDPR)
- **Generate Security Report**

---

### **Phase 3: AI-Driven Security Insights**

- **Analyze Prowler output** with AI tool (custom script/ChatGPT/LLM API)
- Generate:
  - Top 10 security risks
  - AI-suggested remediation steps
  - Risk categorization: Critical, High, Medium, Low

---

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
