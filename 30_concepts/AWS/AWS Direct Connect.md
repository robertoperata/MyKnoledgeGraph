---
tags: 
feature: 
type: 
author: 
source: 
---
# AWS Direct Connect

**AWS Direct Connect (DX)** is a dedicated, private network connection between your on-premises data centre and AWS, bypassing the public internet entirely.

## How it works

- A physical cross-connect is established at an **AWS Direct Connect location** (colocation facility)
- Your network connects to the AWS router via a **dedicated port** (1 Gbps or 10 Gbps) or through a **hosted connection** via a partner (from 50 Mbps up to 10 Gbps)
- One or more **Virtual Interfaces (VIFs)** are created on top of the physical connection:
  - **Private VIF**: access resources in a VPC via private IPs
  - **Public VIF**: access AWS public services (S3, DynamoDB…) without traversing the internet
  - **Transit VIF**: connect to a Transit Gateway

## Key characteristics

- **Consistent, low-latency** throughput — not subject to internet congestion
- **Not encrypted by default** — combine with a VPN over the DX connection (MACsec or IPsec) for encryption
- **Not redundant by itself** — provision two connections or use a VPN as a backup for HA

## Direct Connect Gateway

A **Direct Connect Gateway** allows a single DX connection to reach VPCs across **multiple regions** and accounts, avoiding the need for one DX connection per region.

## Resilience options

| Model | Description |
|---|---|
| Single connection | No redundancy — single point of failure |
| Dual connections, same location | Protects against device failure, not location failure |
| Dual connections, different locations | Maximum resilience — protects against location outage |
| DX primary + VPN backup | Cost-effective HA with acceptable fallback latency |
