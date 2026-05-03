---
title: AWS Security Groups e Network ACLs
type: concept
tags: [thread3, aws]
sources: [[courses/KodeKloud AWS Solutions Architect Associate Certification]], [[courses/hands-on-with-aws-vpcs/knowledge]], [[courses/aws-security-deep-dive-vpcs/knowledge]]
updated: 2026-05-03
related: [[concepts/aws-vpc-fundamentals]], [[concepts/aws-ddos-protection]]
---

# AWS Security Groups e Network ACLs

## Definizione

Sono i due livelli di firewall in AWS che implementano il concetto di **defense in depth** a livello di rete:
- **Security Group (SG)**: firewall stateful a livello di singola istanza
- **Network ACL (NACL)**: firewall stateless a livello di subnet intera

Usarli entrambi — non sono alternativi ma complementari.

## Come funzionano

### Security Groups

**Scope**: applicati a singole **interfacce di rete** (ENI) — non a subnet. Ogni EC2 può avere fino a **5 SG** associati; i loro allow si sommano.

**Stateful**: se permetti traffico outbound, la risposta è automaticamente permessa inbound anche senza regola esplicita. Il SG traccia lo stato delle connessioni.

**Solo allow rules** — modello whitelist (implicit deny all). Non esistono deny rules. Tutto ciò che non è esplicitamente permesso è negato.

```
Security Group Rules:
- ALLOW SSH inbound from 0.0.0.0/0
- ALLOW all outbound
(tutto il resto: DENY implicito)
```

**Referencing SG come source**: invece di CIDR, si può usare un altro SG come sorgente.
```
SG "sensitive-data": ALLOW SSH inbound from SG "bastion-host"
```
Ogni istanza nel SG "bastion-host" può accedere. Scalabile automaticamente.

**Conflitti tra SG**: impossibili — solo allow rules, le regole si sommano. L'unico "conflitto" è con il NACL (vedi sotto).

**Default outbound**: tutto permesso. **Default inbound**: tutto negato.

**Analogia (Rick Crisci — micro-segmentation)**: ogni negozio del centro commerciale ha il suo addetto alla sicurezza. Anche se hai passato la guardia all'ingresso (NACL), ogni negozio ha le sue regole.

### Network ACLs

**Scope**: applicati a **subnet intere**. Ogni subnet ha esattamente 1 NACL; più subnet possono condividere la stessa NACL.

**Stateless**: le regole si applicano **indipendentemente** in entrambe le direzioni. Se permetti outbound, devi avere una regola inbound separata per la risposta — incluse le **ephemeral ports** (1024-65535).

**Allow e Deny rules** — a differenza dei SG. Le regole sono numerate e valutate in **ordine numerico crescente**: la prima che corrisponde vince, le successive non vengono valutate.

```
NACL Inbound:
- Regola 100: ALLOW TCP porta 443 from 0.0.0.0/0
- Regola 200: DENY 0.0.0.0/0
- *: DENY (implicito finale)
```

**Default NACL** (creato con la VPC): permette tutto inbound e outbound.
**Nuovo NACL**: nega tutto per default.

**Best practice naming**: rinominare il NACL allow-all come "allow-all-traffic" — visibile a colpo d'occhio che non filtra nulla.

**Analogia (Rick Crisci)**: la guardia all'ingresso del centro commerciale. Ferma le persone prima di entrare nell'edificio (prima del SG).

### Confronto

| Caratteristica | Security Group | Network ACL |
|---|---|---|
| Scope | Singola interfaccia/istanza | Intera subnet |
| Stateful/Stateless | **Stateful** | **Stateless** |
| Tipi di regole | Solo ALLOW | ALLOW e DENY |
| Ordine regole | Tutte valutate insieme | In ordine numerico, prima match vince |
| Numero per risorsa | Fino a 5 per interfaccia | 1 per subnet |
| Default (nuovo) | Deny all inbound, allow all outbound | Deny all (se nuovo) |
| Regole per risposta | Non servono (stateful) | Servono esplicitamente |

### Come interagiscono

Il traffico passa attraverso **entrambi**: NACL prima (subnet), poi SG (istanza).

```
Traffico inbound:
Internet → IGW → NACL (inbound rules) → SG (inbound rules) → EC2

Traffico outbound:
EC2 → SG (outbound rules) → NACL (outbound rules) → IGW → Internet
```

**Se SG permette SSH ma NACL nega SSH**: il traffico non arriva mai al SG — viene bloccato al subnet level. Il SG non viene valutato.

### Micro-segmentation

I Security Groups implementano **micro-segmentation**: ogni server/VM ha il proprio firewall individuale. Anche un sistema già dentro la VPC non può connettersi liberamente ad altri — deve passare attraverso il SG della destinazione.

Contrapposto al modello tradizionale dove il firewall era solo tra subnet: una volta dentro la subnet, accesso libero a tutti i sistemi.

## Quando usarlo

**Security Groups**: sempre, su ogni istanza. Regole minime necessarie — implicit deny all.

**NACLs**: layer aggiuntivo, soprattutto per:
- Blocco esplicito di IP malevoli (DENY rule — impossibile nei SG)
- Defense in depth con team di gestione separato
- Compliance che richiede controllo a livello subnet

**Two-team approach (Rick Crisci)**: team diversi gestiscono SG e NACL. Se un application owner configura erroneamente un SG che permette tutto, il NACL del team network potrebbe fermare l'errore. Riduce la probabilità di human error composto.

## Bastion Host — Pattern per accesso a istanze private

**Problema**: come gestire (SSH/RDP) un EC2 in subnet privata da internet?

**Soluzione**:
```
Internet
    │ SSH — solo da IP fidati (SG)
    ▼
[Bastion Host] (public subnet, SG "bastion-host")
    │ SSH (key pair)
    ▼
[EC2 privata] (private subnet, SG "sensitive-data" ← source: SG "bastion-host")
```

**Configurazione SG**:
1. SG "bastion-host": ALLOW SSH inbound from `<IP aziendale>/32`
2. SG "sensitive-data": ALLOW SSH inbound from SG "bastion-host"

**Alternativa moderna**: AWS Systems Manager Session Manager — elimina bastion host e porta 22, richiede IAM role e SSM agent.

## Trade-off e limitazioni

- SG: nessun deny esplicito — per bloccare IP malevoli serve il NACL
- NACL stateless: ephemeral ports vanno configurate esplicitamente lato outbound
- NACL ordine numerico: una regola errata con numero basso può bloccare tutto
- SG max 5 per interfaccia: in ambienti complessi può diventare un vincolo

## Connessioni con altri concetti

- [[concepts/aws-vpc-fundamentals]] — i SG e NACL operano all'interno della VPC
- [[concepts/aws-ddos-protection]] — i SG sono l'ultimo layer di difesa prima delle istanze
