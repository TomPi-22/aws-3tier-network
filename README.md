# AWS 3-Tier Segmented Network

**Author:** Tom Cocker

---

## Overview

This repository contains a brief configuration guide for a segmented three-tier network built in AWS. The guide demonstrates subnetting, layered traffic control, least-privilege access, and segmented traffic logging. This deployment was accomplished in an AWS Academy Learner Lab with the use of System Manager rather than SSH access. 

> **Note:** AWS Academy Learner Lab permissions did not allow for modification or creation of new roles. This limits the user's ability to alter AWS IAM controls.

The network was constructed using a single VPC divided into three subnets (web, app, and data). Traffic control was enforced independently at both the NACL and security groups levels. All subnet traffic was captured via VPC flow logs. 

---

## Network Architecture

The environment consists of a single VPC (`vpc-3tier`, `10.0.0.0/16`) split into three subnets within a single Availability Zone (`us-west-2a`):

**Subnets:**
- **subnet-web (`10.0.1.0/24`)** — A public-facing tier intended for a running web server. Reachable on ports 80 and 443. Routes to internet via an internet gateway. Additionally, it hosts the NAT gateway utilized by the app tier.
- **subnet-app (`10.0.2.0/24`)** — Internal application tier. Reachable only from the web tier on port 8080. Outbound-only internet access via NAT Gateway.
- **subnet-data (`10.0.3.0/24`)** — Database tier. Reachable only from the app tier on port 3306. No route to the internet.

**Traffic control:**
- **Security groups** enforce instance-level, stateful rules, chained by reference (`web-sg` → `app-sg` → `data-sg`) instead of the IP range.
- **Network ACLs** enforce subnet-level, stateless rules as a second layer.
- **No SSH access is permitted in this environment.** All instance management is performed via AWS Systems Manager Session Manager.

**Logging:** VPC Flow Logs are enabled on all three subnets and delivered to an S3 bucket, capturing ACCEPTED or REJECTED traffic.

---

## Repository Structure

This repository contains a single deployment guide (`aws-3tier-deployment.md`) and an `images` folder containing all screenshots referenced within it.

---

## Document Summary

### AWS 3-Tier Deployment Guide
Covers the full design and deployment of the segmented network described above. Includes VPC and subnet creation, Internet Gateway and NAT Gateway configuration, route table design isolating the data tier, security group and NACL rule sets for all three tiers, IAM role configuration under AWS Academy Learner Lab constraints, AWS Systems Manager setup including VPC interface endpoints for the internet-isolated data tier, and VPC Flow Logs configuration delivering to S3. A testing and verification section confirms tier isolation, permitted inter-tier traffic, blocked SSH access, and functional traffic logging using live connectivity tests and flow log evidence.

---

## Future Work

- Implementing dedicated least-privilege IAM roles per tier (blocked in this environment by AWS Academy Learner Lab restrictions)
- Extending the architecture across multiple Availability Zones
- Automating the deployment via Terraform

---

## Resources

The resources below assisted in the creation and development of the setup guides listed above. These sources can also be utilized by users to reference official documentation and alternative guides for troubleshooting purposes.

### AWS Networking
- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)

### AWS Systems Manager
- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)
- [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [Interface VPC Endpoints for Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-create-vpc.html)

### VPC Flow Logs
- [VPC Flow Logs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [Publishing Flow Logs to Amazon S3](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3.html)