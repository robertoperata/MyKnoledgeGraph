---
tags: 
feature: 
type: 
author: 
source: 
---
# CIDR (Classless Inter-Domain Routing)

**CIDR** is a method for allocating IP addresses and defining network boundaries. It replaces the old classful system (Class A/B/C) with a flexible notation that specifies exactly how many bits belong to the network portion of an address.

A CIDR block is written as:
```
<IP address>/<prefix length>
```
The prefix length (the number after `/`) tells you how many bits are **fixed** (the network part). The remaining bits are **free** to assign to hosts.

---

## How the calculation works

An IPv4 address is 32 bits. The prefix length splits those 32 bits into two parts:

```
| ← network bits (fixed) → | ← host bits (free) → |
|       prefix length       |     32 - prefix       |
```

From this you can derive everything:

| Value | Formula |
|---|---|
| **Total addresses** | 2^(32 - prefix) |
| **Usable hosts** | Total addresses − 2 (network address + broadcast) |
| **Subnet mask** | Set the first `prefix` bits to 1, rest to 0 |
| **Network address** | IP AND subnet mask |
| **Broadcast address** | Network address OR (inverted subnet mask) |
| **First usable host** | Network address + 1 |
| **Last usable host** | Broadcast address − 1 |

---

## Step-by-step examples

### Example 1 — `192.168.1.0/24`

**Step 1: how many host bits?**
32 − 24 = **8 host bits**

**Step 2: total addresses**
2⁸ = **256**

**Step 3: subnet mask**
24 bits set to 1 → `11111111.11111111.11111111.00000000` → **255.255.255.0**

**Step 4: network address**
The IP with all host bits set to 0 → **192.168.1.0**

**Step 5: broadcast address**
The IP with all host bits set to 1 → **192.168.1.255**

**Step 6: usable range**
192.168.1.**1** → 192.168.1.**254** = **254 usable hosts**

---

### Example 2 — `10.0.0.0/16`

32 − 16 = **16 host bits** → 2¹⁶ = **65,536 addresses**

Subnet mask: `255.255.0.0`
Network: `10.0.0.0`
Broadcast: `10.0.255.255`
Usable: `10.0.0.1` → `10.0.255.254` = **65,534 hosts**

---

### Example 3 — `172.16.0.0/20`

32 − 20 = **12 host bits** → 2¹² = **4,096 addresses**

Subnet mask: `11111111.11111111.11110000.00000000` → `255.255.240.0`
Network: `172.16.0.0`
Broadcast: `172.16.15.255` (the last 12 bits all set to 1)
Usable: `172.16.0.1` → `172.16.15.254` = **4,094 hosts**

> **How to find the broadcast manually**: take the network address in binary, set all host bits to 1.
> `172.16.0.0` = `...00010000.00000000.00000000`
> 12 host bits set to 1 = `...00010000.00001111.11111111` = `172.16.15.255`

---

### Example 4 — `10.0.1.0/28` (small subnet)

32 − 28 = **4 host bits** → 2⁴ = **16 addresses**

Subnet mask: `255.255.255.240`
Network: `10.0.1.0`
Broadcast: `10.0.1.15`
Usable: `10.0.1.1` → `10.0.1.14` = **14 hosts**

AWS uses /28 as its smallest allowed subnet size inside a VPC.

---

## Quick reference table

| Prefix | Total addresses | Usable hosts | Subnet mask |
|---|---|---|---|
| /16 | 65,536 | 65,534 | 255.255.0.0 |
| /20 | 4,096 | 4,094 | 255.255.240.0 |
| /24 | 256 | 254 | 255.255.255.0 |
| /25 | 128 | 126 | 255.255.255.128 |
| /26 | 64 | 62 | 255.255.255.192 |
| /27 | 32 | 30 | 255.255.255.224 |
| /28 | 16 | 14 | 255.255.255.240 |

---

## Checking if two CIDR blocks overlap

Two blocks overlap if the network address of one falls within the range of the other.

`10.0.0.0/16` covers `10.0.0.0` → `10.0.255.255`
`10.0.1.0/24` covers `10.0.1.0` → `10.0.1.255` → **overlaps** (is fully contained)

`10.0.0.0/24` covers `10.0.0.0` → `10.0.0.255`
`10.1.0.0/24` covers `10.1.0.0` → `10.1.0.255` → **no overlap**

In [[VPC Peering]] and VPC subnets, overlapping CIDR blocks are not allowed and will be rejected.
