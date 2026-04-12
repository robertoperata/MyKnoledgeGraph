---
tags: 
feature: 
type: 
author: 
source: 
---
# VPC Peering

**VPC Peering** is a networking connection between two VPCs that allows traffic to be routed between them using private IP addresses, as if they were in the same network.

## Key characteristics

- Works across **different AWS accounts** and **different regions** (inter-region peering)
- Traffic stays on the **AWS backbone** — never traverses the public internet
- **Non-transitive**: if VPC A peers with VPC B, and VPC B peers with VPC C, A cannot reach C through B. Each peering must be established explicitly.
- CIDR blocks of the two VPCs **must not overlap**

## Setup requirements

1. Create a peering connection request (requester VPC)
2. Accept the peering connection (accepter VPC)
3. Update **route tables in both VPCs** to point the remote CIDR to the peering connection
4. Update **Security Groups / NACLs** to allow the desired traffic

## Limitation: no transitive routing

```
VPC A <--peered--> VPC B <--peered--> VPC C
A cannot reach C — a direct peering A↔C is required
```

For hub-and-spoke topologies with many VPCs, prefer [[Transit Gateway]] over a mesh of peering connections.
