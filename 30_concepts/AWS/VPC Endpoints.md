---
tags: 
feature: 
type: 
author: 
source: 
---
# VPC Endpoints

A **VPC Endpoint** allows resources inside a VPC to communicate with supported AWS services **privately**, without requiring an Internet Gateway, NAT Gateway, VPN, or Direct Connect. Traffic never leaves the AWS network.

## Two types

### Gateway Endpoint
- Supports **S3** and **DynamoDB** only
- Added as a **target in the route table** — no DNS change required
- Free of charge
- Region-scoped (not AZ-specific)

### Interface Endpoint (powered by AWS PrivateLink)
- Supports most AWS services (e.g. SQS, SNS, Secrets Manager, CloudWatch, EC2 API…)
- Creates an **Elastic Network Interface (ENI)** with a private IP in your subnet
- Uses **private DNS** to resolve the service's public hostname to the private ENI IP
- Charged per hour + per GB of data processed

## Why use VPC Endpoints

- Improves **security**: traffic stays within AWS, never exposed to the internet
- Reduces **NAT Gateway costs** for high-volume S3/DynamoDB access from private subnets
- Required in environments with **no internet access** (air-gapped VPCs)

## Gateway vs Interface summary

| | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Services | S3, DynamoDB | Most AWS services |
| Implementation | Route table entry | ENI in subnet |
| Cost | Free | Hourly + data charge |
