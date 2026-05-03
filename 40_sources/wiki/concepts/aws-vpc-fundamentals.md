---
title: AWS VPC — Fondamentali
type: concept
tags: [thread3, aws]
sources: [[courses/KodeKloud AWS Solutions Architect Associate Certification]], [[courses/hands-on-with-aws-vpcs/knowledge]], [[courses/aws-security-deep-dive-vpcs/knowledge]]
updated: 2026-05-03
related: [[concepts/aws-security-groups-nacls]], [[concepts/aws-vpc-connectivity]], [[concepts/twelve-factor]], [[technologies/kubernetes]]
---

# AWS VPC — Fondamentali

## Definizione

Una **VPC (Virtual Private Cloud)** è la rete privata virtuale isolata all'interno di AWS. Nulla può comunicare in entrata o uscita finché non si configurano esplicitamente route table, internet gateway, peering ecc.

**Scope**: regionale — una VPC non attraversa regioni né è AZ-specifica. Gli account di un'AZ non sono visibili dalle altre AZ dello stesso VPC.

## Come funziona

### CIDR e indirizzi IP

Ogni VPC ha un **CIDR block** che definisce il range di IP utilizzabili (RFC1918):
- Range massimo: **/16** (65.536 indirizzi) — non modificabile dopo la creazione
- Range minimo: **/28**
- Si possono aggiungere fino a 5 CIDR secondari
- **Regola pratica**: scegliere `/16` fin dall'inizio; non è modificabile dopo

### Default VPC vs Custom VPC

**Default VPC** (creata automaticamente in ogni regione):
- CIDR: `172.31.0.0/16`
- Una subnet `/20` per ogni AZ
- Internet Gateway già attaccato
- Route table con default route `0.0.0.0/0 → IGW`
- Auto-assign public IP abilitato su tutte le subnet
- NACL allow-all inbound e outbound
- Security group di default

**I 3 Strike della Default VPC (Rick Crisci):**
1. Auto-assign public IP su tutte le subnet
2. Route table punta all'Internet Gateway
3. Network ACL permette tutto il traffico

**Best practice**: eliminare il default VPC da ogni regione appena creato l'account. Si può ricreare con `Actions → Create Default VPC`.

### Subnet

Partizione del CIDR della VPC, locale a una singola **Availability Zone**.

**IP riservati per subnet** (es. `192.168.1.0/24`):
- `.0` — network address
- `.1` — VPC router
- `.2` — DNS AWS
- `.3` — future use AWS
- `.255` — broadcast

**Public vs Private Subnet**:

| Caratteristica | Public | Private |
|---|---|---|
| Route default | → Internet Gateway | → NAT Gateway |
| Auto-assign public IP | Abilitato | Disabilitato |
| Uso tipico | Web server, load balancer, bastion host | Database, app server, backend |

Il 90% dei workload va in subnet private. Subnet pubblica solo per entry point diretti.

**Convention CIDR**: private subnet da `10.x.1.x`, public subnet da `10.x.101.x` — rende visibile il tipo dal CIDR.

### Route Table

Controlla il routing del traffico nella VPC. Ogni subnet è associata a esattamente una route table. Più subnet possono condividere la stessa route table.

**Main Route Table**: assegnata automaticamente a ogni nuova subnet. Best practice: mantenerla **"local only"** (solo route locale) — nuove subnet sono automaticamente private.

**Local route**: sempre presente, non modificabile. Permette comunicazione intra-VPC.

**Default route** (`0.0.0.0/0`): catch-all — traffico senza corrispondenza specifica. Punta a IGW (subnet pubblica) o NAT Gateway (subnet privata).

**Most specific route wins**: una route `/24` batte `/16` batte `/0`.

### Internet Gateway (IGW)

Punto di uscita della VPC verso internet. Managed service AWS.
- **1 IGW per VPC** — limite fisso
- Una subnet diventa "pubblica" solo se la sua route table ha `0.0.0.0/0 → igw-xxxx`
- Fa NAT in background: il Public IP non esiste sull'istanza ma sull'IGW

Flusso traffico outbound (public subnet):
```
EC2 (private IP) → SG → NACL → Route Table → IGW → Internet
```

### NAT Gateway

Permette alle istanze in **subnet private** di accedere a internet (aggiornamenti OS, API calls) **senza essere raggiungibili dall'esterno**.

**Posizionamento**: sempre in una **subnet pubblica** (ha bisogno dell'IGW). Richiede un **Elastic IP**. AZ-specifico: se la subnet pubblica cade, si perde il NAT Gateway.

```
EC2 privata → Route Table → NAT GW (EIP pubblico) → IGW → Internet
```

**NAT Gateway (managed) vs NAT Instance (EC2 unmanaged)**:
- NAT Gateway: zero manutenzione, autoscaling, costo ~$32/mese + $0.045/GB
- NAT Instance: meno costoso, più flessibile (usabile come bastion/VPN), gestione OS a tuo carico

Per gli esami: usare sempre NAT Gateway.

### Elastic IP

IP pubblico statico associabile a istanze EC2 o ENI.
- Dinamico (default): cambia ad ogni stop/start — gratuito
- Elastic IP: fisso, spostabile tra istanze — **$0.005/ora sempre**, anche fermo

**Quando usarlo**: DNS record che punta all'IP, failover tra istanze.

### DNS in VPC

- Ogni istanza ottiene automaticamente un hostname DNS privato
- DNS server AWS: `169.254.169.253` o IP VPC + 2
- `enableDnsHostNames`: abilita hostname DNS pubblici per istanze con public IP
- `enableDNSSupport`: abilita la risoluzione DNS via resolver AWS

## Quando usarlo

- Qualsiasi workload AWS — la VPC è il prerequisito per quasi tutti i servizi
- Isolamento tra ambienti (dev/test/prod): VPC separate per isolamento completo
- Connessione a on-premises: la VPC è il punto di attacco per Direct Connect e VPN

## Architettura VPC sicura — Pattern base

```
VPC 10.1.0.0/16
│
├── Route Table: local-only [MAIN]
│   └── (nuove subnet sicure per default)
│
├── Route Table: private-nat-gw → 0.0.0.0/0 → NAT GW
│   ├── Private Subnet AZ1A (10.1.1.0/24)
│   └── Private Subnet AZ1B (10.1.2.0/24)
│
├── Route Table: public-igw → 0.0.0.0/0 → IGW
│   ├── Public Subnet AZ1A (10.1.101.0/24) — auto-assign IP ON
│   └── Public Subnet AZ1B (10.1.102.0/24) — auto-assign IP ON
│
├── Internet Gateway
└── NAT Gateway (in Public Subnet AZ1A + EIP)
```

## Trade-off

- CIDR fisso dopo la creazione — pianificare con `/16`
- 1 IGW per VPC — non diversificabile per subnet
- NAT Gateway: AZ-specifico — per HA servono NAT Gateway in ogni AZ
- Single VPC: facilità di gestione ma meno isolamento rispetto a multi-account

## Connessioni con altri concetti

- [[concepts/aws-security-groups-nacls]] — i firewall che proteggono le risorse nella VPC
- [[concepts/aws-vpc-connectivity]] — come connettere VPC tra loro e a on-premises
- [[concepts/twelve-factor]] — la VPC è il "backing service" per app cloud-native
- [[technologies/kubernetes]] — i cluster EKS girano su subnet VPC
