---
tags:
feature:
type:
author:
source:
---
# NAT Gateway

A **NAT Gateway (Network Address Translation Gateway)** allows resources in a **private subnet** to initiate outbound traffic to the internet, while preventing the internet from initiating inbound connections to those resources.

## How it works

- Deployed in a **public subnet** with an Elastic IP assigned
- Private subnet route table points `0.0.0.0/0` to the NAT Gateway
- The NAT Gateway translates the private IP to its own public Elastic IP before forwarding traffic to the Internet Gateway

## Key characteristics

- **Managed by AWS** — no patching or administration required
- **Highly available** within a single AZ; deploy one per AZ for full HA
- Supports **TCP, UDP, ICMP**
- Cannot be used as a bastion host or for inbound traffic

## NAT Gateway vs NAT Instance

| | NAT Gateway | NAT Instance |
|---|---|---|
| Management | Fully managed by AWS | Self-managed EC2 |
| Availability | Highly available within AZ | Single point of failure |
| Bandwidth | Up to 100 Gbps | Limited by instance type |
| Cost | Higher | Lower (but operational overhead) |
