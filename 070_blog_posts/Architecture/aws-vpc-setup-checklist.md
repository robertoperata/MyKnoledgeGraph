---
tags:
  - aws
  - vpc
  - cloud
  - infrastructure
  - networking
feature:
type: checklist
author: Roberto Perata
source:
date: 2026-04-12
---

# AWS VPC — To-Do List di Setup

## 1. VPC
- Crea VPC con CIDR `10.0.0.0/16`

---

## 2. Internet Gateway
- Crea Internet Gateway
- **Aggancia** l'IGW al VPC appena creato (`Attach to VPC`)

---

## 3. Subnets
- Crea **Public Subnet** con CIDR `10.0.1.0/24` → assegnala al VPC
- Crea **Private Subnet** con CIDR `10.0.2.0/24` → assegnala al VPC
- Abilita **Auto-assign public IPv4** sulla Public Subnet

---

## 4. Elastic IP
- Alloca un **Elastic IP** (serve prima del NAT Gateway)

---

## 5. NAT Gateway
- Crea **NAT Gateway** nella **Public Subnet** (non nella private!)
- **Aggancia** l'Elastic IP allocata al punto 4

---

## 6. Route Tables

**Route Table pubblica:**
- Crea una Route Table → assegnala al VPC
- Aggiungi route: `0.0.0.0/0` → **Internet Gateway**
- **Associa** la Route Table alla **Public Subnet**

**Route Table privata:**
- Crea una seconda Route Table → assegnala al VPC
- Aggiungi route: `0.0.0.0/0` → **NAT Gateway**
- **Associa** la Route Table alla **Private Subnet**

---

## 7. Security Groups

**SG Bastion Host** (Public Subnet):
- Inbound: SSH porta 22 dal tuo IP
- Outbound: tutto

**SG App Server** (Private Subnet):
- Inbound: SSH porta 22 solo dal SG del Bastion Host
- Inbound: porta applicazione (es. 8080) dal Bastion o da un ALB
- Outbound: tutto

**SG RDS** (Private Subnet):
- Inbound: porta DB (es. 5432 PostgreSQL) solo dal SG dell'App Server
- Outbound: tutto

---

## 8. EC2 — Bastion Host
- Lancia EC2 nella **Public Subnet**
- Assegna il **SG del Bastion Host**
- Scegli o crea una **Key Pair** per SSH

---

## 9. EC2 — App Server
- Lancia EC2 nella **Private Subnet**
- Assegna il **SG dell'App Server**
- Verifica che **non** abbia IP pubblico (raggiungibile solo via Bastion)

---

## 10. RDS
- Crea un **DB Subnet Group** con almeno 2 AZ che includano la Private Subnet
- Crea istanza RDS nella **Private Subnet** usando il DB Subnet Group
- Assegna il **SG per RDS**
- Disabilita **Publicly Accessible**

---

## Verifica finale

| Test | Come |
|------|------|
| Bastion raggiungibile | `ssh -i key.pem ec2-user@<elastic-ip-bastion>` |
| App Server via Bastion | `ssh -J ec2-user@<bastion> ec2-user@<ip-privato-app>` |
| App Server ha accesso internet | `curl https://example.com` dall'App Server |
| RDS raggiungibile dall'App Server | `psql -h <rds-endpoint> -U user -d db` |
| RDS non raggiungibile da internet | connessione diretta deve fallire |

---

## Riferimenti
- [[Excalidraw/AWS VPC Network Overview]]
