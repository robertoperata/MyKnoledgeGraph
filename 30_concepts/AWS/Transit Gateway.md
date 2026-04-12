---
tags: 
feature: 
type: 
author: 
source: 
---
# Transit Gateway

**AWS Transit Gateway (TGW)** is a regional network hub that allows you to connect multiple VPCs, AWS accounts, VPNs, and Direct Connect gateways through a single gateway, avoiding a full mesh of VPC peering connections.

## Problem it solves

With VPC Peering, connecting N VPCs requires up to N×(N-1)/2 peering connections. Transit Gateway introduces a **hub-and-spoke model**: every VPC connects to the TGW, and the TGW handles routing between them.

```
VPC A ──┐
VPC B ──┤── Transit Gateway ──── VPN / Direct Connect
VPC C ──┘
```

## Key characteristics

- **Transitive routing**: unlike VPC Peering, a packet can traverse from VPC A → TGW → VPC B
- Supports **inter-region peering** between Transit Gateways
- Each attachment (VPC, VPN, Direct Connect) has its own route table on the TGW
- **Multicast** support (unique among AWS networking constructs)
- Charged per attachment-hour + per GB of data processed

## Route tables on TGW

TGW has its own route tables independent of VPC route tables. You can segment traffic by associating attachments to different TGW route tables (e.g., production VPCs isolated from dev VPCs).

## When to use vs VPC Peering

| Scenario | Use |
|---|---|
| 2–3 VPCs, simple connectivity | VPC Peering |
| Many VPCs, hub-and-spoke, VPN/DX integration | Transit Gateway |
