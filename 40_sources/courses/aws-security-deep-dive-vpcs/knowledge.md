---
titolo: AWS Security Deep Dive: VPCs, Networking, and DDoS Mitigation
piattaforma: O'Reilly
istruttore: Rick Crisci
data: 2024-11-10
durata_totale: 4h
trascrizione_coperta: 4h su 4h totali
lingua_originale: en
iterazione: 1
tags:
  - aws
  - vpc
  - networking
  - security
  - cloud
  - ddos
  - waf
  - direct-connect
---

# AWS Security Deep Dive: VPCs, Networking, and DDoS Mitigation

## Chi è l'istruttore

Rick Crisci è il proprietario di **Trainertest.com**, una piattaforma di online learning e preparazione esami. Ha creato oltre 30 corsi e insegnato a più di 250.000 studenti. I suoi corsi sono presenti su Pearson, LinkedIn Learning e Udemy. È co-autore del libro *AWS Certified SysOps Administrator Associate Exam Cram*. Facilitatrice O'Reilly: Brittany Brady.

---

## A chi è rivolto e obiettivi dichiarati

**Target:** professionisti con una conoscenza base di AWS e VPC, sia chi prepara certificazioni (Solutions Architect Associate, SysOps Administrator, Network Specialty, Architect Professional) sia chi usa AWS in produzione. Circa due terzi dei partecipanti preparavano una certificazione.

**Prerequisiti dichiarati:** aver seguito "Hands-on with AWS VPCs" di Rick Crisci.

**Obiettivi:** imparare a costruire reti AWS sicure — VPC con subnets private/public, security groups, NACLs, bastion host, VPN, Direct Connect, VPC peering, VPC endpoints, DDoS mitigation con WAF, Shield, CloudFront.

---

## Struttura del corso

| Modulo | Titolo | Durata stimata | Coperto |
|--------|--------|----------------|---------|
| 1 | VPC Security Fundamentals — Default VPC e costruzione sicura | ~1h | ✅ |
| 2 | Bastion Host, Security Groups e NACLs in dettaglio | ~1h | ✅ |
| 3 | VPC Peering, VPC Endpoints, VPN, Direct Connect | ~1h | ✅ |
| 4 | DDoS Mitigation (Shield, WAF, CloudFront, Route 53) | ~1h | ✅ |

> Pause registrate: 62-72 min (10 min) e 120-130 min (10 min) — pause reali tra moduli, non silenzio del recording.

---

## Contesto e motivazione

La maggior parte delle violazioni di sicurezza AWS deriva da **configurazioni errate**, non da vulnerabilità sofisticate. Il default AWS è orientato alla facilità d'uso, non alla sicurezza. Rick Crisci costruisce il corso attorno a un principio fondamentale:

> *"Se diamo alle persone la possibilità di fare le cose in modo sbagliato, le faranno in modo sbagliato."*

L'approccio è invertire il default: rendere la configurazione sicura quella automatica, e richiedere sforzo esplicito per aprire permessi.

---

## Concetti fondamentali

### Default VPC — I 3 Strike

Ogni account AWS riceve automaticamente un **Default VPC** in ogni regione. Rick lo chiama "security nightmare" per tre ragioni precise:

**Strike 1 — Auto-assign Public IP**
Le subnet del default VPC hanno l'opzione "Auto-assign public IP" abilitata. Ogni EC2 instance creata ottiene automaticamente un IP pubblico raggiungibile da internet.

**Strike 2 — Route Table punta all'Internet Gateway**
La route table principale del default VPC ha una default route (`0.0.0.0/0`) che punta direttamente all'Internet Gateway. Le istanze possono uscire su internet senza NAT.

**Strike 3 — Network ACL permissiva**
Il Network ACL associato alle subnet del default VPC permette tutto il traffico in entrata e in uscita (`ALLOW ALL` inbound, `ALLOW ALL` outbound).

**Raccomandazione pratica:** eliminare il default VPC da ogni regione subito dopo aver creato l'account. AWS spaventa con un warning ma si può ricreare in 5 secondi con "Actions → Create Default VPC".

---

### Main Route Table e Default Network ACL

**Main Route Table:** è la route table assegnata automaticamente a ogni nuova subnet creata nella VPC. Se la main route table ha una route verso internet, tutte le nuove subnet erediteranno quell'accesso.

**Default Network ACL:** è il NACL applicato automaticamente alle nuove subnet. Nel default VPC è `ALLOW ALL`.

**Best practice di Rick:** rendere la main route table **"local only"** e il default NACL il più restrittivo possibile. Così chi crea una subnet per semplicità ottiene qualcosa di sicuro, non qualcosa di aperto.

---

### Subnet Privata vs Pubblica

La differenza non è un flag nativo — è una combinazione di configurazioni:

| Caratteristica | Subnet Privata | Subnet Pubblica |
|----------------|---------------|-----------------|
| Auto-assign public IP | Disabilitato | Abilitato |
| Route table default route | → NAT Gateway | → Internet Gateway |
| IP delle istanze | Solo privato | Pubblico + Privato |
| Raggiungibilità da internet | No | Sì |

**Convention CIDR di Rick:** private subnets a partire da `10.x.1.x`, public subnets a partire da `10.x.101.x`. Rende immediatamente visibile dal CIDR se una subnet è privata o pubblica.

**Raccomandazione:** il 90%+ dei workload dovrebbe stare in subnet private. Solo gli entry point pubblici (load balancer, bastion host) stanno in subnet pubbliche.

---

### Internet Gateway (IGW)

L'IGW permette la comunicazione tra la VPC e internet. Da solo non è sufficiente — serve una route table che lo indichi come destinazione del traffico. Un IGW attaccato a una VPC non espone nulla automaticamente.

---

### NAT Gateway

Il NAT Gateway permette alle risorse in **subnet private** di iniziare connessioni verso internet (aggiornamenti software, API calls) senza essere raggiungibili dall'esterno.

```
EC2 privata (10.1.1.10)
    │
    ▼
Route Table: 0.0.0.0/0 → NAT Gateway
    │
    ▼
NAT Gateway (IP pubblico EIP: 54.x.x.x)
    │
    ▼
Internet Gateway → Internet
```

**NAT Gateway vs NAT Instance:** NAT Gateway è il servizio gestito (recommended) e fa dynamic many-to-one NAT. Una NAT Instance su EC2 offre NAT one-to-one statico o configurazioni custom, ma richiede gestione manuale (patching, security groups).

---

### Routing in AWS — Regole chiave

1. **Most specific route wins:** una route `/24` batte una `/16` batte una `/0`.
2. **Default route (`0.0.0.0/0`):** fallback — intercetta tutto il traffico senza route più specifica.
3. **Local route:** ogni VPC ha automaticamente una route locale non modificabile.
4. **Una subnet = un route table:** ogni subnet è associata a esattamente una route table. Più subnet possono condividere la stessa route table.
5. **Route Propagation:** in ambienti ibridi con VPN o Direct Connect, le route BGP possono essere propagate automaticamente nelle route table.

---

### Security Groups — Firewall a Livello Istanza

I Security Groups sono firewall applicati **a singole istanze EC2** (o interfacce di rete). Caratteristiche fondamentali:

**Stateful:** il SG traccia lo stato delle connessioni. Se un'istanza invia una richiesta outbound e il SG permette il traffico outbound, il traffico di ritorno (response) viene automaticamente permesso anche se non c'è una regola inbound esplicita.

**Solo allow rules:** non esistono deny rules nei Security Groups. È un modello **whitelist** (implicit deny all). Tutto ciò che non è esplicitamente permesso è negato.

**Fino a 5 SG per interfaccia:** un'istanza EC2 può avere più SG associati. Ogni SG aggiunge allow rules — la combinazione è l'unione di tutte le allow rules.

**Referencing SG come source:** invece di specificare IP/CIDR come sorgente, si può specificare un altro SG. Esempio: il SG "sensitive-data" permette SSH solo dalla SG "bastion-host". Ogni EC2 in "bastion-host" può connettersi a "sensitive-data", scalabile automaticamente.

**Conflitti tra SG:** non esistono conflitti — solo allow rules, quindi si sommano senza contraddizioni.

**Conflitto SG vs NACL:** se il SG permette SSH inbound ma il NACL nega SSH inbound, SSH è bloccato — il traffico non raggiunge mai il SG.

**Analogia:** i SG sono come ogni negozio dentro un centro commerciale che ha il suo addetto alla sicurezza (micro-segmentation). Anche se passi la guardia all'ingresso (NACL), ogni negozio ha le sue regole.

---

### Network ACLs — Firewall a Livello Subnet

I Network ACLs (NACLs) sono firewall applicati a **interi subnet**. Caratteristiche fondamentali:

**Stateless:** non tracciano lo stato delle connessioni. Servono regole esplicite sia per il traffico inbound che per il traffico outbound, incluso il traffico di ritorno. Questo include la gestione delle **ephemeral ports** (porte temporanee usate per le risposte, tipicamente 1024-65535).

**Allow e Deny rules:** a differenza dei SG, i NACLs supportano sia allow che deny. Le regole sono numerate e valutate in ordine crescente. La prima regola che matcha vince.

**Una NACL per subnet:** ogni subnet ha esattamente una NACL associata. Più subnet possono condividere la stessa NACL.

**NACL di default:** permette tutto il traffico in entrata e in uscita. Rick suggerisce di rinominarla "allow-all-traffic" per rendere visibile la mancanza di protezione.

**Raccomandazione team separati:** Rick suggerisce che in organizzazioni strutturate il team che gestisce i Security Groups dovrebbe essere diverso da chi gestisce i NACLs. Se un application owner configura erroneamente un SG che permette tutto, il NACL potrebbe fermare l'errore.

**Analogia:** il NACL è come la guardia all'ingresso del centro commerciale. Ferma le persone prima di entrare nell'edificio.

---

### Micro-segmentation

Concetto di sicurezza in cui ogni server/VM ha il proprio firewall (Security Group). Contrapposto al modello tradizionale in cui il firewall era solo tra subnet: una volta dentro la subnet, si poteva accedere a qualsiasi sistema liberamente.

Con i Security Groups, anche un sistema già all'interno della VPC non può connettersi liberamente ad altri — deve passare attraverso il SG dell'istanza di destinazione.

---

### Naming Convention — Sicurezza by Design

Rick è molto diretto su questo: la **chiarezza nei nomi riduce errori di configurazione**.

- Subnet: `private-subnet-az1a`, `public-subnet-az1b`
- Route table: `local-only` (per il main), `private-nat-gw`, `public-igw`
- NACL: `allow-all-traffic` (così chi guarda sa subito che è permissiva)
- IGW: `rick-demo-igw`

---

### Bastion Host / Jump Box

**Problema:** come gestire (SSH/RDP) un'EC2 in subnet privata da internet?

**Soluzione:** **Bastion Host** — un'EC2 in subnet pubblica con IP pubblico, con Security Group che permette SSH (porta 22) solo da IP fidati (es. IP aziendale). Si entra nel bastion, poi dal bastion ci si connette all'istanza privata usando la key pair.

```
Internet
    │ SSH (porta 22) — solo da IP fidati
    ▼
[Bastion Host] (subnet pubblica, IP pubblico)
    │ SSH (key pair)
    ▼
[EC2 privata] (subnet privata, solo IP privato)
```

**Demo completa:**
1. Crea EC2 bastion in public subnet con SG "allow-ssh-inbound"
2. Crea EC2 "sensitive-data" in private subnet con SG "private-instances"
3. Configura SG "private-instances": allow SSH source = SG "bastion-host"
4. Connettiti al bastion → dal bastion usa il key pair per SSH all'istanza privata

**Nota:** EC2 instance connect non funziona su istanze senza public IP. Per accedere senza bastion, serve AWS Systems Manager Session Manager (installa agent, configurazione IAM).

**Alternative moderne:**
- **AWS Systems Manager Session Manager** — elimina bastion e porta 22, richiede IAM role e SSM agent
- **EC2 Instance Connect** — solo per istanze con public IP

---

### VPC Isolation e VPC Peering

**Default:** ogni VPC è completamente isolata. Non può comunicare con altre VPC, con internet, con VPN o con qualsiasi altro sistema — a meno che non si configuri esplicitamente.

**Ragioni per avere più VPC:** separare ambienti (dev/test/prod), separare team/dipartimenti, isolare workload sensibili.

**VPC Peering (PCX):** connessione diretta tra due VPC che permette il routing del traffico tra di esse. Il traffico non passa per internet.

**Steps per configurare VPC Peering:**
1. **Genera richiesta di peering** dalla VPC sorgente verso la VPC destinazione (può essere stesso account, stesso region; oppure account diverso o region diversa)
2. **Accetta la richiesta** dal proprietario della VPC destinazione
3. **Aggiorna le route table in entrambe le VPC**: ogni subnet che deve comunicare deve avere una route che indica la destinazione (CIDR dell'altra VPC) e il target (il PCX)

**Requisito fondamentale:** i CIDR delle due VPC **non devono sovrapporsi**. Se si sovrappongono, serve NAT (molto più complesso).

**Sicurezza:** VPC peering non bypassa Security Groups e NACLs — questi continuano ad applicarsi normalmente.

---

### VPC Endpoints

**Problema:** S3 e DynamoDB non vivono dentro la VPC. Il traffico da EC2 a S3 deve percorrere internet (IGW → internet → S3), con costi di trasferimento dati e rischi di sicurezza.

**Soluzione — Gateway Endpoint (gratuito):** per S3 e DynamoDB. Si aggiunge una route nella route table che dice "per traffico verso S3/DynamoDB, usa l'endpoint" — il traffico rimane sulla rete AWS senza toccare internet.

**Soluzione — Interface Endpoint (a pagamento):** per la maggior parte degli altri servizi AWS. Crea un'interfaccia di rete privata nella subnet con un IP privato. Il traffico verso il servizio AWS usa quell'IP privato.

**Benefici:** sicurezza (traffico non esposto a internet), costo (transfer charges più bassi), performance (latenza ridotta).

---

### VPN Site-to-Site

Connessione cifrata tra data center on-premises e VPC AWS.

**Opzione 1 — Virtual Private Gateway (VGW) — Managed:**
```
Data Center → Customer Gateway → Internet → VGW → VPC
```
- AWS gestisce il VGW (managed service)
- Supporta BGP e routing dinamico
- Alta disponibilità gestita da AWS
- Non si controlla l'endpoint AWS — AWS lo gestisce

**Opzione 2 — Software VPN su EC2 — Unmanaged:**
```
Data Center → Internet Gateway → EC2 (software VPN) → VPC
```
- Si controlla entrambi gli endpoint VPN
- Pro: controllo totale (configurazione custom, visibilità completa)
- Contro: bisogna gestire patching, security groups, NACLs, disponibilità dell'EC2
- Analogia: "costruisci una casa ultra-sicura ma poi dimentichi di chiudere la porta"

**Per gli esami AWS:** se la domanda chiede VPN site-to-site, la risposta standard è Virtual Private Gateway.

---

### Direct Connect

Connessione **fisica dedicata** tra data center on-premises e AWS backbone. Non passa per internet.

**Architettura:**
```
Data Center → [AT&T/ISP] → Co-location Facility → AWS Router → AWS Backbone → VPC
     Router              Router (your)  Router (AWS)
```

**Steps:**
1. Installa il tuo router nella co-location facility
2. Compra un circuito fisico (AT&T, Spectrum, MPLS) tra data center e co-location
3. Il datacenter della co-location esegue un **cross-connect** tra il tuo router e il router AWS
4. Configura un **Virtual Interface (VIF)** sul lato AWS
5. Aggiorna le route table in VPC e nel data center

**Tipi di Virtual Interface:**
- **Private VIF:** connette al tuo VPC — per workload privati
- **Public VIF:** connette a servizi pubblici AWS (S3, DynamoDB) senza passare per internet
- **Transit VIF:** connette a Transit Gateway (per connettere più VPC)

**Velocità:** 1 Gbps, 10 Gbps, 100 Gbps.

**Sicurezza:** il traffico non passa per internet — maggiore controllo e sicurezza. Si può aggiungere VPN over Direct Connect per cifrare il traffico (doppio strato).

**Costo vs VPN:** Direct Connect sembra più caro (router, co-location, circuito) ma i **transfer charges** su Direct Connect sono significativamente inferiori a quelli su VPN/internet. Per grandi volumi di dati, Direct Connect può essere più economico.

> **Per esami AWS:** assumere che Direct Connect sia più economico del VPN per workload con grandi trasferimenti di dati.

---

### WAF Sandwich (Firewall EC2 Customizzati)

Pattern architetturale per chi vuole usare firewall di terze parti (Fortinet, Palo Alto, Juniper, Checkpoint, Barracuda, PFSense — disponibili nel marketplace EC2).

```
Internet
    │
[NLB — Internet Facing]  ← riceve tutto il traffico
    │
[EC2 Firewall 1] [EC2 Firewall 2] [EC2 Firewall 3]  ← auto scaling group
    │
[Internal ALB]
    │
[Web Server 1] [Web Server 2] [Web Server 3]  ← private subnet, no public IP
```

**Benefici:** pieno controllo sul firewall, familiarità con tool esistenti del data center, auto scaling dei firewall stessi.

**Downside:** bisogna gestire il patching e la configurazione degli EC2 firewall.

---

### AWS WAF (Web Application Firewall)

Firewall gestito da AWS, integrato in:
- **CloudFront**
- **Application Load Balancer (ALB)**
- **API Gateway**

**Modalità di configurazione:**
1. **Managed Rules:** AWS configura e aggiorna le regole automaticamente — opzione "hands-off"
2. **Custom Rules:** si definiscono condizioni specifiche (IP block lists, geo-blocking, SQL injection detection, ecc.)
3. **Rate-based Rules:** limitazione di velocità — se un singolo IP supera X richieste in Y secondi, viene bloccato o de-prioritizzato

**Rate-based rules contro DDoS:**
- Il WAF identifica le sorgenti ad alto traffico
- De-prioritizza le loro richieste
- Il traffico legittimo continua a passare
- Il blocco avviene "al bordo" (in CloudFront), prima che il traffico raggiunga i web server

---

### CloudFront + Route 53 — Security Architecture

**CloudFront** è la CDN di AWS con edge location in tutto il mondo. Funziona come primo livello di difesa:

- Risponde alle richieste HTTP/HTTPS direttamente dall'edge (cache)
- Assorbe HTTP floods, SYN floods, TLS request floods senza che il traffico raggiunga i web server
- AWS WAF è integrato — filtra richieste maligne all'edge
- Nasconde il vero IP dei web server

**Route 53:**
- Gestisce il DNS e reindirizza il traffico a CloudFront
- AWS Shield Standard è built-in su Route 53
- Route 53 è così massivamente scalabile da essere praticamente immune a DDoS volumetrici

**Architettura completa di difesa:**
```
Internet
    │
[Route 53] ← Shield Standard, DNS-level protection
    │
[CloudFront] ← AWS WAF, DDoS absorption, HTTP flood protection
    │
[ALB] ← AWS WAF, Security Groups
    │
[EC2 Web Servers] ← Security Groups, NACLs, auto scaling
```

**Principio:** "obfuscation" — gli attaccanti attaccano AWS (CloudFront, Route 53) invece di attaccare direttamente i server che gestisci tu. AWS ha capacità di assorbimento degli attacchi enormemente superiore.

---

### AWS Shield

**AWS Shield Standard (gratuito):**
- Automaticamente incluso per tutti i clienti AWS
- Protegge Route 53, CloudFront, ALB, ELB
- Protegge contro attacchi layer 3/4 comuni: SYN floods, UDP floods, reflection attacks

**AWS Shield Advanced (a pagamento):**
- Protezione avanzata per EC2, ELB, CloudFront, Route 53, Global Accelerator
- Supporto del team AWS DDoS Response Team (DRT)
- Copertura dei costi AWS aggiuntivi durante un attacco DDoS
- Dashboard e metriche dettagliate sugli attacchi

---

### Auto Scaling come Mitigazione DDoS

Approccio controverso ma valido: invece di bloccare il DDoS, **si scala orizzontalmente** per assorbire il traffico.

**Calcolo di Rick:** 1000 istanze T3.large × $0.08/ora × 24 ore = ~$1.920. Se i danni di un giorno di downtime superano $2K, conviene pagare.

**Logica:** gli attacchi DDoS non riusciti durano pochi minuti a poche ore. Se l'attacco non funziona, gli attaccanti si spostano su un target più facile.

**Criticità:** molte organizzazioni temono la bolletta. Ma il costo del downtime è solitamente superiore.

---

### AWS Well-Architected Framework — Security Pillar

Citato da Rick come riferimento. Il principio chiave per la sicurezza:

> *"Place your vulnerable systems behind AWS managed services."*

AWS vuole che gli attaccanti attacchino **AWS** (CloudFront, Route 53, ALB) invece delle risorse che gestisci tu. Questi managed services hanno capacità di assorbimento degli attacchi enormi e baked-in DDoS protection.

---

### AWS Macie e GuardDuty (menzione Q&A)

**AWS Macie:** servizio ML-driven che:
- Scopre automaticamente dati sensibili in S3 (PII, dati finanziari, credenziali)
- Fornisce alerting automatico in caso di anomalie di accesso
- Categorizza e protegge dati sensibili

**AWS GuardDuty:** intelligent threat detection per account AWS:
- Monitora continuamente account, workload, traffico di rete
- Usa ML per identificare pattern anomali
- Rileva: credenziali compromesse, mining di criptovalute, comunicazioni verso C2, privilege escalation
- Integra threat intelligence AWS + third-party

---

## Architettura e design

### Architettura VPC Sicura — Pattern Base

```
                        INTERNET
                            │
                    [Internet Gateway]
                            │
         ┌──────────────────┼──────────────────┐
         │              VPC 10.1.0.0/16          │
         │                  │                   │
         │    Route Table: public-igw            │
         │    (0.0.0.0/0 → IGW)                 │
         │         │               │            │
         │  [Public Subnet AZ1A]  [Public Subnet AZ1B]
         │  10.1.101.0/24         10.1.102.0/24  │
         │  Auto-assign IP: ON                   │
         │   [Bastion Host]                      │
         │         │                             │
         │  [NAT Gateway + EIP]                  │
         │         │                             │
         │    Route Table: private-nat-gw        │
         │    (0.0.0.0/0 → NAT GW)              │
         │         │               │            │
         │  [Private Subnet AZ1A] [Private Subnet AZ1B]
         │  10.1.1.0/24           10.1.2.0/24   │
         │  Auto-assign IP: OFF                  │
         │   [EC2 App]  [EC2 DB]                 │
         └────────────────────────────────────────┘
```

### Architettura DDoS-Resiliente con WAF e CloudFront

```
                        INTERNET
                            │
                     [Route 53]  ← Shield Standard + DNS
                            │
                    [CloudFront]  ← AWS WAF + Shield + caching
                            │
              [Application Load Balancer]  ← AWS WAF + SG
                            │
         ┌──────────────────┼──────────────────┐
         │              VPC (private subnets)   │
         │                                     │
         │  [EC2 Web 1]  [EC2 Web 2]  [...]   │
         │   ← Auto Scaling Group              │
         │   ← Security Groups + NACLs         │
         └─────────────────────────────────────┘
                            │
                     [S3 Bucket]  ← assets/static content
```

### Architettura Direct Connect

```
Data Center                    Co-location Facility              AWS
    │                               │                             │
[Router] ──── AT&T Circuit ────> [My Router] ─cross-connect─> [AWS Router]
                                                                  │
                                                          AWS Backbone
                                                                  │
                                                           [VPC/S3/DynamoDB]

Private VIF → accede a VPC
Public VIF  → accede a S3/DynamoDB (servizi pubblici AWS)
VPN over Direct Connect → cifra tutto il traffico
```

---

## Demo e esempi pratici

### Demo: Eliminazione e Ricreazione Default VPC

**Cosa mostra:** perché e come eliminare il default VPC.
**Come funziona:**
1. VPC Console → seleziona default VPC → Actions → Delete VPC
2. AWS mostra warning intimidatorio ma reversibile
3. Per ripristinarlo: Actions → Create Default VPC (5 secondi)
**Cosa si impara:** eliminare il default VPC è sicuro e reversibile. Non farlo è lasciare una porta aperta.

---

### Demo: Costruzione VPC Sicura da Zero

**Step-by-step completo:**
1. Crea VPC: `rick-demo`, CIDR `10.1.0.0/16`
2. Crea private subnets (auto-assign IP: OFF):
   - `private-subnet-az1a` → 10.1.1.0/24
   - `private-subnet-az1b` → 10.1.2.0/24
3. Rinomina main route table → `local-only`
4. Rinomina default NACL → `allow-all-traffic`
5. Crea IGW: `rick-demo-igw` → attach alla VPC
6. Crea NAT Gateway: `rick-demo-nat` nella subnet pubblica con EIP
7. Crea route table `private-nat-gw`: route `0.0.0.0/0 → NAT GW` → associa private subnets
8. Crea public subnets (auto-assign IP: ON):
   - `public-subnet-az1a` → 10.1.101.0/24
   - `public-subnet-az1b` → 10.1.102.0/24
9. Crea route table `public-igw`: route `0.0.0.0/0 → IGW` → associa public subnets

---

### Demo: Bastion Host e Accesso a Istanza Privata

**Setup:**
1. Crea EC2 bastion in `public-subnet-az1a`, SG "allow-ssh-inbound" (porta 22)
2. Crea EC2 "sensitive-data" in `private-subnet-az1a`, SG "private-instances"
3. In SG "private-instances": add inbound rule → SSH → source: SG "bastion-host"
4. Connessione: `ssh -i keypair.pem ec2-user@<bastion-public-ip>`
5. Dal bastion: `ssh -i keypair.pem ec2-user@<private-ec2-private-ip>`

**Pattern con backup server:**
- Crea SG "backups"
- Aggiungi SG "backups" all'istanza privata → ora il backup server ha accesso

---

### Demo: VPC Peering

**Setup:**
1. Crea dev VPC: CIDR `192.168.1.0/24`
2. Crea subnet dev: `192.168.1.0/25`
3. In Rick Demo VPC: Peering Connections → Create → seleziona dev VPC → Create
4. In dev VPC: accetta la richiesta di peering
5. In Rick Demo VPC route table: aggiungi `192.168.1.0/24 → PCX-xxxxx`
6. In dev VPC route table: aggiungi `10.1.0.0/16 → PCX-xxxxx`

---

### Demo: WAF Rate-Limiting

**Scenario:** bad guys con botnet tentano DDoS sul web server.
**Configurazione:** AWS WAF con rate-based rule su CloudFront.
**Funzionamento:** WAF osserva il traffico, identifica sorgenti con richieste eccessive, de-prioritizza le loro richieste, il traffico legittimo passa normalmente. L'attacco viene fermato all'edge, mai vicino ai web server.

---

## Trade-off e limitazioni

| Aspetto | Pro | Contro |
|---------|-----|--------|
| Subnet private | Non raggiungibili da internet | Richiedono bastion host o SSM per gestione |
| NAT Gateway | Sicuro, managed, HA | Costo mensile (~$32/mese + trasferimento dati) |
| Main route table "local only" | Nuove subnet sicure di default | Step extra per abilitare internet access |
| Virtual Private Gateway (VPN) | Managed, facile da configurare | Traffico passa per internet (anche cifrato) |
| Software VPN su EC2 | Controllo totale su entrambi gli endpoint | Bisogna gestire patching e vulnerabilità EC2 |
| Direct Connect | Sicuro (no internet), alta banda, costi transfer ridotti | Setup costoso (router, co-location, circuito fisico) |
| VPC Peering | Semplice, traffico non su internet | Non transitivo (A↔B, B↔C non implica A↔C) |
| Gateway VPC Endpoint (S3/DynamoDB) | Gratuito, traffico su backbone AWS | Solo S3 e DynamoDB |
| Interface VPC Endpoint | Supporta molti servizi AWS | A pagamento per ora |
| AWS WAF Managed Rules | Zero configurazione | Meno controllo granulare |
| AWS WAF Custom Rules | Controllo preciso | Richiede expertise e manutenzione |
| WAF Sandwich (EC2 firewall) | Pieno controllo, familiarità con tool on-prem | Bisogna gestire EC2, patching, disponibilità |
| Auto Scaling anti-DDoS | Assorbe l'attacco, sito rimane up | Costo AWS più elevato durante l'attacco |
| Shield Standard | Gratuito, automatico | Protezione base solo layer 3/4 |
| Shield Advanced | Protezione avanzata, supporto DRT | Costo significativo |
| Bastion Host | Semplice, comprensibile | Single point of failure, superficie d'attacco pubblica |
| SSM Session Manager | No porta 22, no IP pubblico | Richiede IAM role e SSM agent su ogni istanza |

---

## Sicurezza e considerazioni operative

- **MFA obbligatoria** su root account e tutti gli IAM user con accesso AWS console
- **Eliminare default VPC** in ogni regione subito dopo la creazione account
- **Non assegnare public IP** a EC2 instance a meno che non strettamente necessario
- **Usare NAT Gateway** invece di IGW diretto per subnet private
- **Main route table = local only:** chi crea subnet per sbaglio non ottiene internet access automatico
- **Default NACL:** rinominarla "allow-all-traffic" per rendere visibile il rischio
- **VPC CIDR non sovrapposti:** critico per VPC peering e Direct Connect
- **Direct Connect è preferibile a VPN** per connessioni ibride ad alto volume
- **Nascondere web server dietro CloudFront + ALB** (obfuscation)
- **AWS WAF con rate-limiting** come prima linea contro DDoS layer 7
- **Defense in depth:** Route 53 → CloudFront → ALB → EC2 con più strati indipendenti
- **Separazione di responsabilità:** team diversi per Security Groups e NACLs
- **Monitoraggio:** GuardDuty per anomalie, Macie per dati sensibili in S3

---

## Q&A notevoli

**Q:** Cos'è la Route Propagation nelle route table?
**A:** Se si usa BGP con VPN o Direct Connect verso il data center on-premises, si possono propagare automaticamente le route BGP nelle route table AWS invece di aggiungerle manualmente.

**Q:** Il NAT Gateway fa NAT statico one-to-one o dynamic many-to-one?
**A:** Il NAT Gateway gestisce dynamic NAT (many-to-one). Per NAT statico serve una NAT Instance (EC2 custom).

**Q:** Con MFA e un attaccante MITM che osserva il codice, MFA è davvero sicura?
**A:** Il MITM è vulnerabilità reale di TOTP MFA. Ma non si abbandona MFA — si minimizza il rischio come si può. Analogia: il casco in moto non elimina il rischio ma lo riduce drasticamente. Passkey/FIDO2 mitigano meglio il MITM.

**Q:** Security Groups vs Network ACLs — quando usare cosa?
**A:** Entrambi sempre. SG = stateful, a livello istanza, solo allow. NACL = stateless, a livello subnet, allow e deny. Due team separati li gestiscono idealmente. NACL non può fermare qualcosa che il SG permette, ma il SG non può fermare qualcosa bloccato dal NACL (il traffico non arriva mai al SG).

**Q:** Come vengono risolti i conflitti tra più Security Groups sullo stesso EC2?
**A:** Non ci sono conflitti — i SG hanno solo allow rules. Il risultato è l'unione di tutte le allow rules di tutti i SG associati.

**Q:** Se SG non ha deny, come blocco traffico che un altro SG permette?
**A:** Non puoi bloccare con i SG — solo il NACL può bloccare esplicitamente. È un limite intenzionale del modello.

**Q:** Un EC2 può avere più Security Groups?
**A:** Sì, fino a 5 SG per interfaccia. Ogni SG aggiunge le sue allow rules.

**Q:** VPC peering è transitivo?
**A:** No. Se VPC A è peered con B, e B è peered con C, A e C non possono comunicare. Ogni peering è una connessione diretta punto-punto.

**Q:** Direct Connect vs VPN — quale è più economico?
**A:** Direct Connect sembra più costoso inizialmente (router, co-location, circuito), ma i transfer charges sono significativamente inferiori. Per grandi volumi di dati, Direct Connect può essere più economico. Per gli esami: Direct Connect è più economico.

**Q:** C'è un GitHub repo per il corso?
**A:** No. Rick ha condiviso link a lab exercises gratuiti su trainertest.com durante il corso (link messi nel chat window, necessario registrarsi).

**Q:** AWS Macie e GuardDuty — cosa fanno?
**A:** Macie: ML-driven per trovare e proteggere dati sensibili in S3. GuardDuty: threat detection intelligente per account AWS, monitora comportamenti anomali, usa ML.

**Q:** Risorse su EKS?
**A:** Rick non ha un corso su EKS e non riesce a raccomandare qualcosa immediatamente.

---

## 📚 Risorse e riferimenti

### Tool e servizi AWS citati
- **AWS Shield Standard** — DDoS protection gratuito incluso in Route 53 e CloudFront
- **AWS Shield Advanced** — DDoS protection avanzato a pagamento con supporto DRT
- **AWS WAF** — Web Application Firewall su CloudFront e ALB, regole managed e custom
- **AWS CloudFront** — CDN + security layer, edge location globali
- **AWS Route 53** — DNS managed + Shield Standard built-in
- **AWS Macie** — ML-driven sensitive data discovery e protection per S3
- **AWS GuardDuty** — Intelligent threat detection per account AWS
- **AWS Systems Manager Session Manager** — alternativa bastion host senza porta 22
- **AWS Well-Architected Framework** — framework di best practice AWS (security pillar: hide behind managed services)

### Firewall di terze parti disponibili in EC2 Marketplace
- Fortinet, Palo Alto Networks, Juniper, Checkpoint, Barracuda, PFSense

### Corsi correlati
- **Hands-on with AWS VPCs** — Rick Crisci — O'Reilly — prerequisito consigliato per questo corso
- **AWS Certified SysOps Administrator - Associate (SOA-C02)** — Chad Smith — O'Reilly
- **AWS Certified Solutions Architect - Associate** — O'Reilly
- **Hands-on with AWS Containers and Fargate** — Rick Crisci — O'Reilly (annunciato durante il corso)
- **Aurora and RDS Databases** — Rick Crisci — O'Reilly (annunciato durante il corso)
- **Perplexity AI Essentials** — Rick Crisci — O'Reilly (annunciato per dicembre)

### Libri
- **AWS Certified SysOps Administrator Associate Exam Cram** — Rick Crisci (co-autore) — preparazione esame SysOps

### Risorse dell'istruttore
- **Trainertest.com** — piattaforma di Rick Crisci con lab exercises
- **Lab exercises VPC** — condivisi durante il corso tramite link trainertest.com: creazione VPC, subnets, IGW, NAT gateway, VPC peering, PrivateLink (non disponibile GitHub repo)
- **LinkedIn di Rick Crisci** — contatto per problemi con lab exercises

### Incidenti di sicurezza citati
- **Healthcare ransomware attack 2024** — organizzazione sanitaria US compromessa perché MFA non abilitata; riscatto ~$20M pagato; dati venduti comunque; costo industria stimato $2-3B

---

## Takeaway applicabili

1. **Elimina i default VPC** in ogni regione — effort: basso — fallo adesso
2. **Abilita MFA** su root account e tutti gli IAM user — effort: basso — immediato
3. **Usa private subnet per default** per tutti i workload applicativi — effort: medio
4. **Imposta la main route table come "local only"** in ogni VPC — effort: basso
5. **Usa naming convention esplicite** per subnet, route table, NACL (ruolo + destinazione traffico) — effort: basso
6. **Usa CIDR separati per subnet private e pubbliche** (es. private: 10.x.1-100.x, public: 10.x.101+.x) — effort: basso
7. **Configura Security Groups per referencing SG invece di CIDR** quando possibile (più scalabile) — effort: basso
8. **Metti i web server dietro CloudFront** per assorbimento DDoS e obfuscation — effort: medio
9. **Abilita AWS WAF su CloudFront/ALB con rate-based rules** come protezione base DDoS layer 7 — effort: basso
10. **Valuta Direct Connect** per connessioni ibride con alto volume di dati — effort: alto — long term
11. **Usa VPC Gateway Endpoint per S3/DynamoDB** — gratuito, traffico su backbone AWS — effort: basso
12. **Abilita GuardDuty** per threat detection continua — effort: basso

---

## Materiali aggiuntivi disponibili

- [x] Lab exercises VPC: condivisi da Rick su trainertest.com durante il corso (link nel chat window O'Reilly, registrazione gratuita richiesta) — include VPC creation, subnets, IGW, NAT gateway, VPC peering, PrivateLink
- [ ] Slide: non disponibili pubblicamente
- [ ] Video on-demand lab demonstrations: disponibile su trainertest.com (a pagamento, ~3.5h di demo)

---

## Note sulla trascrizione

- **Copertura:** 14317s su 14319s totali (~100%) — intero corso coperto
- **Pause non-audio:** 62-72 min e 120-130 min sono pause reali tra sessioni (partecipanti in pausa), non silenzio del recording
- **Qualità trascrizione:** buona. "Rick Rishi" è trascrizione errata di "Rick Crisci". Tutti i termini tecnici AWS trascritti correttamente.
- **Contenuto coperto:** tutti i moduli — VPC fundamentals, bastion host, SG vs NACL, VPC peering, VPC endpoints, VPN, Direct Connect, WAF, Shield, CloudFront, Route 53, DDoS mitigation, GuardDuty, Macie
