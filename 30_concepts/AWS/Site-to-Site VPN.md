---
tags: 
feature: 
type: 
author: 
source: 
---
# Site-to-Site VPN

**AWS Site-to-Site VPN** creates an encrypted IPsec tunnel between your on-premises network and your AWS VPC over the **public internet**.

## Components

- **Virtual Private Gateway (VGW)**: the AWS-side VPN endpoint, attached to the VPC
- **Customer Gateway (CGW)**: a resource in AWS representing your on-premises VPN device (router or firewall), identified by its public IP
- **VPN Connection**: links the VGW to the CGW; each connection has **two tunnels** for redundancy

## Key characteristics

- Encrypted with **IKEv1 or IKEv2 + IPsec**
- Each tunnel: up to **1.25 Gbps** throughput (AWS-side limit)
- Both tunnels should be configured on the customer device to avoid single points of failure
- Supports **static routing** or **dynamic routing via BGP**
- Traffic traverses the **public internet** — latency and bandwidth are not guaranteed

## Accelerated Site-to-Site VPN

An optional variant that routes VPN traffic through **AWS Global Accelerator**, reducing latency by entering the AWS backbone closer to the customer.

## When to use vs Direct Connect

| | Site-to-Site VPN | Direct Connect |
|---|---|---|
| Setup time | Minutes | Weeks to months |
| Cost | Low | Higher (port + partner fees) |
| Bandwidth | Up to ~1.25 Gbps | Up to 100 Gbps |
| Latency | Variable (public internet) | Consistent (dedicated line) |
| Use case | Quick setup, backup link | Predictable performance, large data transfer |
