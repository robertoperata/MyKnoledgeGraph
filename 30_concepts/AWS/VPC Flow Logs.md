---
tags: 
feature: 
type: 
author: 
source: 
---
# VPC Flow Logs

**VPC Flow Logs** capture metadata about IP traffic going to and from network interfaces in a VPC. They do not capture the packet payload — only the connection metadata.

## What is captured

Each flow log record includes:
- Source and destination IP and port
- Protocol
- Bytes and packets transferred
- Action: `ACCEPT` or `REJECT` (based on Security Group / NACL decisions)
- Start and end time

## Scope levels

Flow Logs can be enabled at three levels:
1. **VPC** — captures all traffic across all ENIs in the VPC
2. **Subnet** — captures all traffic for ENIs in that subnet
3. **ENI** — captures traffic for a single network interface

## Destinations

- **CloudWatch Logs** — queryable with Logs Insights
- **S3** — cheaper for long-term retention, queryable with Athena
- **Kinesis Data Firehose** — for real-time streaming to downstream systems

## Common use cases

- **Security forensics**: identify which IPs are hitting your resources and whether traffic is being rejected
- **Troubleshooting connectivity**: confirm whether packets are reaching an ENI and if they are accepted
- **Compliance**: retain network access records for audit purposes

## Limitations

- Not real-time — there is a delay of several minutes
- Does not capture traffic to/from `169.254.169.254` (instance metadata), DNS queries to the VPC resolver, or DHCP traffic
