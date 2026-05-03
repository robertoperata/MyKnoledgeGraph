---
title: AWS VPC Connectivity — Peering, Transit Gateway, Direct Connect, VPN, Endpoints
type: concept
tags: [thread3, aws]
sources: [[courses/hands-on-with-aws-vpcs/knowledge]], [[courses/aws-security-deep-dive-vpcs/knowledge]]
updated: 2026-05-03
related: [[concepts/aws-vpc-fundamentals]], [[concepts/aws-security-groups-nacls]], [[technologies/kubernetes]]
---

# AWS VPC Connectivity

## Definizione

Una VPC è isolata per default — nessuna comunicazione verso altre VPC, internet, o on-premises senza configurazione esplicita. Questo concetto raccoglie i meccanismi per connettere VPC tra loro e con sistemi esterni.

## VPC Peering

Connessione diretta di routing tra **due VPC** che permette comunicazione come se fossero sulla stessa rete. Il traffico non attraversa internet — resta sull'**AWS backbone**.

**Caratteristiche:**
- Funziona cross-account e cross-region
- Limite soft: 50 peering per VPC (aumentabile a 125)
- AWS managed — nessun hardware, alta disponibilità inclusa
- **Non transitivo**: A↔B, B↔C non implica A↔C. Per connettere N VPC serve Transit Gateway

**Requisito critico — CIDR non sovrapposti**: se due VPC hanno range sovrapposti, il routing è impossibile.

**Configurazione:**
1. Crea la peering connection (PCX)
2. Accetta la richiesta (necessario se cross-account)
3. Aggiorna le route table **in entrambe le VPC**: CIDR remoto → PCX

```
Route Table VPC-A: 192.168.1.0/24 → pcx-xxxxx
Route Table VPC-B: 10.1.0.0/16   → pcx-xxxxx
```

SG e NACL continuano ad applicarsi normalmente anche con peering attivo.

**VPC Peering vs VPN fai-da-te**: peering è managed, su backbone, alta disponibilità. VPN su EC2 richiede gestione e cade se l'istanza cade.

## Transit Gateway

**Hub centrale** che risolve il problema del full-mesh con molte VPC. Invece di N×(N-1)/2 peering, ogni VPC si connette al Transit Gateway.

```
VPC-A ─┐
VPC-B ─┤
VPC-C ─┤── Transit Gateway ──── On-prem / VPN / Direct Connect
VPC-D ─┤
VPC-E ─┘
```

**Cosa può connettere (risposta esame: tutti):**
- VPC (qualsiasi account/regione)
- VPN (Customer Gateway on-prem)
- Direct Connect Gateway (DXGW)
- Altri Transit Gateway (multi-regione)

**Costi:** $0.05/ora per ogni attachment + $0.02/GB traffico.

**Transit VPC (approccio legacy):** router di terze parti (Cisco, Palo Alto, Fortinet) su EC2 usato come hub. Ancora utile per deep packet inspection layer 7 su tutto il traffico inter-VPC.

**Esame tip**: Transit Gateway = "ho molte VPC". VPC Peering = "ho 2 VPC".

## VPC Endpoints

Permettono comunicazione tra risorse VPC e servizi AWS **senza che il traffico attraversi internet**.

**Problema**: S3 e DynamoDB non vivono dentro la VPC. Senza endpoint, `EC2 → S3` passa per internet — costi di egress e rischio sicurezza.

### Gateway Endpoint (gratuito)
- Solo per **S3 e DynamoDB**
- Aggiunge automaticamente una route table entry con un prefix list (IP di S3/DynamoDB nella regione)
- Zero costo aggiuntivo — traffico sull'AWS backbone
- Creato di default dal wizard "VPC and more"

### Interface Endpoint (billable)
- Per 280+ servizi AWS (EC2 API, SSM, CloudWatch, Secrets Manager ecc.)
- Crea un'ENI nella subnet con IP privato
- Non richiede modifiche alla route table
- Costo: ore + GB

### PrivateLink
Meccanismo che abilita gli interface endpoint. Permette anche di **esporre un proprio servizio** ad altri account tramite endpoint — il traffico non esce mai su internet.

```
Account A: NLB → servizio (avatar generator)
Account B: interface endpoint → accede al servizio di A via PrivateLink
```

## VPC Flow Logs

Registrano metadati sul traffico IP che transita attraverso le interfacce di rete. Analogo a NetFlow.

**Configurazione:**
- Livello: VPC, subnet, o singola ENI
- Filtro: accepted, rejected, o tutto
- Destinazione: S3 o CloudWatch Logs

**Formato:** interfaccia, IP src/dst, porta, protocollo, bytes, stato (ACCEPT/REJECT).

**Uso:** troubleshooting (perché il traffico non arriva?), security anomaly detection, compliance.

## VPN Site-to-Site (Managed)

Connessione cifrata IPsec tra data center on-premises e VPC AWS, passando per internet.

**Componenti:**
- **Virtual Private Gateway (VGW)**: lato AWS — diverso dall'IGW
- **Customer Gateway (CGW)**: rappresentazione del router on-prem in AWS
- **CPE (Customer Premises Equipment)**: device fisico on-prem con Public IP

**Configurazione:**
```
VPC → VGW ─(IPsec over internet)─ CPE (Public IP) → On Prem
```

**Caratteristiche:**
- Setup rapido (ore)
- IPsec — tutto cifrato
- Costo egress: **$0.09/GB**
- Performance variabile (dipende da internet)

**Software VPN**: alternativa su EC2. Controllo totale (compliance), ma gestione OS/HA a carico tuo. Analogia: "costruisci casa super-sicura ma dimentichi di chiudere la porta."

## Direct Connect

Connessione **fisica dedicata** tra data center on-premises e AWS backbone. Non passa per internet.

**Architettura:**
```
Data Center → [Circuito ISP: 1G/10G/100G] → Co-location Facility
                                               ├── Il tuo router
                                               └── AWS router → AWS Backbone → VPC
```

**Steps per attivarlo:**
1. Acquista circuito fisico (AT&T, Spectrum ecc.) verso Direct Connect Location
2. Installa il tuo router nella co-location facility
3. Richiedi ad AWS il cross-connect tra i due router
4. Configura Virtual Private Gateway (VGW) nella VPC
5. Configura BGP routing

**Virtual Interface (VIF):**
- **Private VIF**: connette al tuo VPC — per workload privati
- **Public VIF**: connette a servizi pubblici AWS (S3, DynamoDB) senza internet
- **Transit VIF**: connette a Transit Gateway

**Direct Connect Gateway (DXGW):**
Hub che permette a **un singolo circuito fisico** di raggiungere **più VPC** (anche in regioni diverse):
```
On Prem → DXGW → VPC1, VPC2, VPC3, VPC4, VPC5
```

**Benefici:**
- Throughput predicibile, latenza bassa e costante
- Egress: **$0.02/GB** (vs $0.09/GB del VPN)
- Traffico non esposto a internet
- VPN over Direct Connect: aggiunge crittografia IPsec sopra, gratuito

**Costo vs VPN**: Direct Connect sembra più costoso (router, co-location, circuito), ma per grandi volumi il risparmio sull'egress ($0.07/GB × volume) può superare il costo fisso.

> **Per gli esami AWS**: assumere che Direct Connect sia più economico del VPN per workload con grandi trasferimenti di dati.

## Data Transfer Charges — Riepilogo

| Tipo traffico | Costo |
|---|---|
| Ingress verso AWS | Gratuito |
| Egress via internet (da EC2) | $0.05–$0.09/GB |
| Egress via Direct Connect | $0.02/GB |
| Egress via VPN | $0.09/GB |
| Tra AZ diverse (stessa regione) | $0.01/GB |
| Stessa AZ | Gratuito |
| Cross-region | Sempre a pagamento |
| Via NAT Gateway | A pagamento |
| Via VPC Endpoint S3/DynamoDB (stessa regione) | Gratuito |

**Trappola comune**: EC2 in AZ-a con RDS in AZ-b → paga $0.01/GB. "Stessa regione ≠ gratuito."

## Confronto VPN vs Direct Connect

| Aspetto | Managed VPN | Direct Connect |
|---|---|---|
| Setup | Ore | Settimane/mesi |
| Throughput | Variabile | Predicibile |
| Latenza | Variabile | Bassa/costante |
| Costo egress | $0.09/GB | $0.02/GB |
| Encryption | IPsec incluso | Opzionale (gratuito) |
| Costo fisso | Nessuno | Circuito + port fee |

## Quando usarlo

- **VPC Peering**: 2 VPC da connettere direttamente
- **Transit Gateway**: molte VPC (>3) o connessione centralizzata on-prem
- **Gateway VPC Endpoint**: accesso a S3/DynamoDB dalla VPC — sempre abilitare
- **Interface VPC Endpoint**: accesso a servizi AWS sensibili senza internet
- **VPN**: connessione on-prem rapida, volume dati moderato
- **Direct Connect**: connessione on-prem con alto volume dati o requisiti di latenza/sicurezza

## Connessioni con altri concetti

- [[concepts/aws-vpc-fundamentals]] — la VPC è il punto di attacco per tutte queste connessioni
- [[concepts/aws-security-groups-nacls]] — SG e NACL si applicano anche al traffico tra VPC peered
- [[technologies/kubernetes]] — i cluster EKS multi-account usano Transit Gateway o VPC Peering
