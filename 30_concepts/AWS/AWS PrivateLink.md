---
tags: 
feature: 
type: 
author: 
source: 
---
# AWS PrivateLink

**AWS PrivateLink** is the underlying technology that powers **Interface VPC Endpoints**. It allows you to expose a service hosted in your VPC (or an AWS service) to other VPCs **privately**, without VPC peering, Internet Gateways, or public IPs.

## How it works

- The **service provider** creates a **Network Load Balancer (NLB)** in front of their service and registers it as an **endpoint service**
- The **service consumer** creates an **Interface Endpoint** in their VPC pointing to that endpoint service
- An **ENI** with a private IP is placed in the consumer's subnet
- Traffic flows privately over the AWS backbone through the NLB to the provider's service

## Key characteristics

- **No CIDR overlap constraint** — unlike VPC Peering, overlapping IP ranges are not a problem
- **Unidirectional**: the consumer can reach the provider, but not vice versa
- Supports **cross-account** and **cross-region** (with some limitations)
- Used by AWS itself to expose services like S3, SQS, SSM as Interface Endpoints

## Use case

Exposing a SaaS service or internal platform service to hundreds of consumer VPCs without full network peering — each consumer gets a private ENI, no route table changes beyond that.
