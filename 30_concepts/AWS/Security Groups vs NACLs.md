---
tags: 
feature: 
type: 
author: 
source: 
---
# Security Groups vs NACLs

Both **Security Groups (SGs)** and **Network Access Control Lists (NACLs)** are firewalls in AWS VPC, but they operate at different layers and have different behaviours.

## Comparison

| | Security Group | NACL |
|---|---|---|
| Applies to | ENI (instance level) | Subnet boundary |
| State | **Stateful** — return traffic is automatically allowed | **Stateless** — return traffic must be explicitly allowed |
| Rules | Allow rules only — no explicit deny | Allow and **deny** rules |
| Rule evaluation | All rules evaluated together | Rules evaluated **in order** by rule number; first match wins |
| Default | Deny all inbound, allow all outbound | Allow all inbound and outbound |
| Association | Attached to one or more ENIs | One NACL per subnet; one subnet per NACL |

## Stateful vs Stateless explained

- **Security Group (stateful)**: if you allow inbound TCP port 443, the response traffic on the ephemeral port is automatically allowed out, with no additional outbound rule needed.
- **NACL (stateless)**: you must explicitly allow both the inbound request (port 443) and the outbound response (ephemeral ports 1024–65535) in separate rules.

## Layered defence

They are complementary, not alternatives. A typical setup:
- **NACLs** as a coarse subnet-level guard (e.g., block a known malicious IP range)
- **Security Groups** as the fine-grained per-resource firewall (e.g., allow only port 8080 from the load balancer SG)
