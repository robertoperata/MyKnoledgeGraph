---
titolo: Hands-on with AWS VPCs
piattaforma: O'Reilly Live Events
istruttore: Rick Crisci
data: 2025-08-23
durata_totale: 6h (Giorno 1 + Giorno 2 — corso completo)
trascrizione_coperta: 6h su 6h (100%)
lingua_originale: en
iterazione: 3
tags: [aws, vpc, networking, cloud, security-groups, network-acl, nat-gateway, subnets, vpc-peering, transit-gateway, load-balancer, vpc-endpoints, direct-connect, vpn, flow-logs, iac, cloudformation, terraform, direct-connect-gateway, vif, internal-elb, route53]
---

# Hands-on with AWS VPCs — Corso Completo (Giorno 1 + Giorno 2)

## Chi è l'istruttore

Rick Crisci è stato **AWS Official Instructor** per anni, insegnando direttamente per AWS e per VMware. La sua specializzazione è virtualizzazione e cloud computing. Ha lasciato la docenza ufficiale per creare corsi propri, con un focus maggiore sugli aspetti **hands-on** rispetto ai corsi ufficiali AWS, che riteneva carenti dal punto di vista pratico. È attivo su LinkedIn.

---

## A chi è rivolto e obiettivi dichiarati

**Target**: chi vuole imparare AWS VPC in profondità, dai principianti a chi prepara certificazioni (Cloud Practitioner, Solutions Architect Associate, SysOps/Cloud Ops Engineer, Networking Specialty).

**Prerequisiti**: accesso a un account AWS su cui sperimentare. Consigliato usare la regione **North California (us-west-1)**.

**Obiettivo**: al termine del corso (2 giorni) si sarà in grado di progettare, creare e gestire VPC AWS in modo sicuro e conforme alle best practice.

**Nota certificazioni**: Rick menziona che l'esame SysOps Administrator Associate (SOA-C02) sta per essere sostituito dal nuovo **AWS Certified Cloud Ops Engineer** (previsto rilascio fine settembre/inizio ottobre del 2025).

---

## Struttura del corso

| Giorno | Argomenti principali | Coperto |
|--------|---------------------|---------|
| 1 — Fondamentali | Default VPC, CIDR, subnets, route tables, IGW, NAT Gateway, Security Groups, Network ACLs, multi-VPC | ✅ |
| 2 — Avanzato | VPC Peering, Transit Gateway, Load Balancers, VPC Endpoints, VPC Flow Logs, VPN, Direct Connect, Data Transfer Charges, IaC (CloudFormation/Terraform) | ✅ |

---

## Concetti fondamentali

### VPC (Virtual Private Cloud)

Una VPC è la **rete privata virtuale** all'interno di AWS. Ogni VPC è isolata: nulla può comunicare in entrata o uscita finché non si aggiungono esplicitamente route table entries, internet gateway, peering ecc.

**Scope**: una VPC è **regionale** — non globale, non AZ-specifico. Una VPC in North California non è disponibile in us-east-1.

**Limite CIDR**: il range di indirizzi IP deve essere uno dei range privati RFC1918. Il range massimo è **/16** (65.536 indirizzi), il minimo è **/28**. Una volta impostato, **non può essere modificato** — scegliere la dimensione giusta fin dall'inizio.

**Best practice CIDR**: non sovrapporre con altri network che si controllano (es. data center on-premise). Due reti con range sovrapposti non possono comunicare tra loro tramite routing.

### Default VPC

Quando si crea un account AWS, ogni regione ha già una **Default VPC** creata automaticamente. È progettata per semplicità, non per sicurezza.

**Perché la Default VPC è insicura — i 3 "strike":**
1. **Strike 1** — Tutte le subnet hanno una route verso l'Internet Gateway: il traffico può uscire e entrare da/verso internet.
2. **Strike 2** — Le subnet assegnano automaticamente **Public IP** a ogni istanza creata.
3. **Strike 3** — Il Network ACL permette **tutto il traffico** in ingresso e uscita (regola: allow all, 0.0.0.0/0).

**Raccomandazione di Rick**: appena si crea un account AWS, eliminare subito tutte le Default VPC prima che qualcuno inizi a usarle. È possibile ricrearle con `Actions → Create Default VPC`.

Non è possibile eliminare una VPC che contiene risorse attive (EC2, RDS ecc.) — va svuotata prima.

### Subnet

Una subnet è una **partizione del CIDR della VPC**. È locale a una singola **Availability Zone** — non può estendersi su più AZ.

**Scope**: AZ-specifico.

**Best practice**: creare subnet in almeno 2 AZ diverse per garantire **alta disponibilità**. Se az1a fallisce, az1c continua a servire il traffico.

**Ogni subnet è associata a:**
- Una **Route Table** (che governa il routing del traffico)
- Un **Network ACL** (che governa le regole firewall a livello di subnet)

### Public vs Private Subnet

| Caratteristica | Public Subnet | Private Subnet |
|---|---|---|
| Definizione AWS | Ha una route verso l'Internet Gateway | Non ha route verso l'IGW |
| Auto-assign Public IP | Abilitato | Disabilitato |
| Uso tipico | Web server, load balancer, bastion host | Database, application server, backend |
| Accesso internet in uscita | Diretto via IGW | Tramite NAT Gateway |

**Regola pratica**: il 90% dei workload va in subnet private. Si usa una subnet pubblica solo se serve accesso diretto da/verso internet.

### Route Table

La route table è come il **GPS del traffico** nella VPC: ogni entry dice "per questo range di IP, manda il traffico a questo target."

**Main Route Table**: creata automaticamente con la VPC. È la **default** per tutte le nuove subnet. Contiene sempre almeno la **local route** (traffico interno alla VPC).

**Local route**: sempre presente, non modificabile. Permette la comunicazione intra-VPC.

**Default route (0.0.0.0/0)**: "catch-all" — tutto il traffico che non corrisponde ad altre entry. Usata per mandare traffico a Internet Gateway o NAT Gateway.

**Best practice di Rick**: tenere la Main Route Table con **solo la local route** (nome: "default-local-only"). Così ogni nuova subnet è automaticamente privata. Creare una route table separata per le subnet pubbliche (nome: "internet-access") con la default route verso l'IGW.

### Internet Gateway (IGW)

L'Internet Gateway è il **punto di uscita della VPC verso internet**. È un servizio managed di AWS.

- Una VPC può avere **solo 1 IGW**
- Una subnet diventa "pubblica" solo quando la sua route table ha una entry `0.0.0.0/0 → igw-xxxx`
- L'IGW fa **NAT** in background per le istanze con Public IP: il Public IP non esiste sull'istanza stessa ma sull'IGW

**Flusso traffico outbound (public subnet)**:
```
EC2 (private IP) → Security Group → Network ACL → Route Table → IGW → Internet
```

### NAT Gateway

Il NAT Gateway permette alle istanze in **subnet private** di accedere a internet (es. per aggiornamenti OS) **senza essere raggiungibili dall'esterno**.

**Come funziona**:
```
EC2 (private IP) → Route Table → NAT GW (sostituisce private IP con public IP) → IGW → Internet
```

**Posizionamento**: sempre in una **subnet pubblica** (ha bisogno dell'IGW per uscire su internet). Richiede un **Elastic IP**.

**NAT Gateway vs NAT Instance**:

| | NAT Gateway | NAT Instance |
|---|---|---|
| Tipo | Managed (AWS gestisce OS, patch, scaling) | Unmanaged (EC2 — tu gestisci tutto) |
| Scalabilità | Automatica | Manuale |
| Manutenzione | Zero | A tuo carico |
| Costo | Più costoso | Meno costoso |
| Flessibilità | Limitata | Puoi usarla come bastion host, VPN, ecc. |

**Raccomandazione AWS (e per gli esami)**: usare sempre il **NAT Gateway** (servizio managed).

### Public IP vs Elastic IP

| | Automatically Assigned Public IP | Elastic IP |
|---|---|---|
| Costo | Gratuito | $0.005/ora (sempre, anche se non usato) |
| Staticità | Dinamico: cambia ad ogni stop/start | Statico: fisso finché non lo rilasci |
| Portabilità | Legato all'istanza | Spostabile tra istanze |

**Quando usare l'Elastic IP**: quando si ha bisogno di un IP pubblico che non cambi (es. DNS record che punta all'IP). Può essere ri-associato a una nuova istanza in caso di failure.

**Elastic Network Interface (ENI)**: come l'Elastic IP, ma per l'intera interfaccia di rete. Si può spostare da un'istanza a un'altra.

### Security Group

Il Security Group è un **firewall stateful** applicato a livello di **singola interfaccia di rete** (non al subnet).

**Caratteristiche chiave:**
- **Stateful**: se permetti traffico outbound, la risposta è automaticamente permessa inbound (anche se non c'è regola inbound esplicita)
- Solo regole **ALLOW** (non esistono regole DENY — tutto ciò che non è esplicitamente permesso è implicitamente negato)
- **Default outbound**: tutto permesso
- **Default inbound**: tutto negato
- Si possono applicare **più security group** a una stessa istanza (l'istanza eredita tutte le regole ALLOW da tutti i SG)
- Scope: **regionale** (ma i nuovi "shared security groups" possono coprire più VPC e account)
- Una regola può usare un altro **security group come sorgente/destinazione** (non solo IP ranges)

### Network ACL (NACL)

Il Network ACL è un **firewall stateless** applicato a livello di **subnet intera**.

**Caratteristiche chiave:**
- **Stateless**: le regole si applicano **indipendentemente** in entrambe le direzioni. Se permetti traffico outbound, devi avere una regola inbound separata per la risposta
- Contiene sia regole **ALLOW che DENY**
- Le regole vengono processate **in ordine numerico** — la prima che corrisponde viene applicata, le successive non vengono valutate
- **Default NACL** (creato con la VPC): permette tutto inbound e outbound
- **Nuovo NACL**: nega tutto per default (implicit deny finale)
- Scope: **subnet all'interno della VPC** (può applicarsi a più subnet, ma non attraverso regioni)

**Esempio ordine regole:**
- Regola 120: DENY HTTP outbound → applicata per prima
- Regola 150: ALLOW HTTP outbound → mai raggiunta se il traffico corrisponde alla 120

### Analogia Castello (Security Groups + NACLs)

```
Internet → [Muro = AWS Shield] → [Fossato = Network ACL] → [Ponte levatoio = Security Group] → EC2
```
- **AWS Shield** (muro esterno): protezione DDoS
- **Network ACL** (fossato): prima linea di difesa a livello subnet
- **Security Group** (ponte levatoio): ultima linea di difesa a livello istanza

---

## Architettura costruita durante il corso

```
AWS Region: us-west-1 (North California)
│
└── VPC: rick-demo (10.1.0.0/16)
    │
    ├── Route Table: internet-access (default route → IGW)
    │   ├── Public Subnet az1a (10.1.1.0/24) → auto-assign public IP
    │   └── Public Subnet az1c (10.1.2.0/24) → auto-assign public IP
    │
    ├── Route Table: main-nat-gw (default route → NAT GW)
    │   ├── Private Subnet az1a (10.1.101.0/24)
    │   └── Private Subnet az1c (10.1.102.0/24)
    │
    ├── Internet Gateway: rick-demo-igw (attached alla VPC)
    │
    ├── NAT Gateway: rick-demo-nat-gw
    │   └── In: Public Subnet az1a | Elastic IP allocato
    │
    ├── Network ACL: default-allow-all (sulle public subnet)
    │   └── Regola 100: ALLOW all inbound/outbound
    │
    ├── Security Group: web-servers
    │   ├── Inbound: SSH (porta 22) from 0.0.0.0/0
    │   └── Inbound: HTTP (porta 80) from 0.0.0.0/0
    │
    └── EC2: web-server
        ├── Subnet: Public az1a
        ├── AMI: Amazon Linux
        ├── Tipo: t3.micro (free tier)
        ├── Public IP: auto-assigned (dinamico)
        ├── User Data: installa Apache, crea pagina "Hello AWS Students"
        └── Security Group: web-servers
```

---

## Demo e esempi pratici

### Demo 1 — Esplora Default VPC
Navigare nella console AWS → VPC Dashboard → VPCs per vedere la Default VPC. Verificare: CIDR /16, subnets per ogni AZ, route table con IGW, Network ACL allow-all, auto-assign public IP abilitato.

### Demo 2 — Elimina e ricrea Default VPC
Selezionare Default VPC → Actions → Delete VPC. Per ricrearla: Actions → Create Default VPC.

### Demo 3 — Crea VPC manuale vs "VPC and more"
AWS offre l'opzione **"VPC and more"** che crea automaticamente VPC + subnets + route tables + NAT Gateway con un wizard. Rick la confronta al "saper guidare una macchina" vs "essere meccanico". Per il corso si crea tutto manualmente per capire i componenti.

### Demo 4 — Crea VPC
VPC Dashboard → Create VPC → nome "rick-demo", CIDR 10.1.0.0/16. Questo crea automaticamente una Main Route Table e un Network ACL di default.

### Demo 5 — Crea Internet Gateway e attacha alla VPC
Internet Gateways → Create → "rick-demo-igw" → Attach to VPC → rick-demo.

### Demo 6 — Crea Route Table pubblica
Route Tables → Create → "internet-access" → VPC: rick-demo → Edit Routes → Add: 0.0.0.0/0 target IGW.

### Demo 7 — Crea subnets pubbliche
Create Subnet → VPC: rick-demo → nome "public-subnet-az1a" → AZ: us-west-1a → CIDR: 10.1.1.0/24. Ripetere per az1c con CIDR 10.1.2.0/24. Poi per ciascuna: Edit subnet settings → Enable auto-assign public IPv4.

### Demo 8 — Associa route table pubblica alle subnet pubbliche
Per ogni subnet pubblica: Route Table tab → Edit route table association → seleziona "internet-access".

### Demo 9 — Crea subnets private
Stesse operazioni, CIDR: 10.1.101.0/24 (az1a) e 10.1.102.0/24 (az1c). NON abilitare auto-assign public IP. Restano associate alla Main Route Table (local only).

### Demo 10 — Crea NAT Gateway
NAT Gateways → Create → "rick-demo-nat-gw" → Subnet: public-subnet-az1a → Elastic IP: Allocate. Poi aggiornare la Main Route Table: aggiungere 0.0.0.0/0 → NAT GW. Rinominare la route table "main-nat-gw".

### Demo 11 — Crea EC2 Web Server
EC2 → Launch Instance → nome "web-server" → AMI: Amazon Linux → t3.micro → Key pair: crea nuovo "rick-demo-august-2025" → Network: rick-demo → Subnet: public-subnet-az1a → Auto-assign public IP: Enable → Security Group: crea "web-servers" con regole SSH + HTTP → User Data:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello AWS Students</h1>" > /var/html/www/index.html
```

### Demo 12 — Verifica connettività
SSH nell'istanza → `ifconfig` (mostra solo private IP, il public IP "vive" sull'IGW) → `curl google.com` (verifica accesso internet).

### Demo 13 — Comportamento Public IP dinamico
Stoppare l'istanza → il Public IP scompare. Riavviarla → ottiene un nuovo Public IP diverso.

### Demo 14 — Elastic IP
Elastic IPs → Allocate Elastic IP Address → associa all'istanza. L'IP è ora fisso. Può essere spostato su un'altra istanza. **Sempre billable** anche quando l'istanza è ferma.

### Demo 15 — Network ACL personalizzato
Network ACLs → Create → "demo-network-acl" → Inbound: regola 150 ALLOW HTTP → Outbound: regola 150 ALLOW HTTP. Dimostrazione ordine regole: regola 120 DENY HTTP outbound ha precedenza su regola 150 ALLOW HTTP outbound.

---

## Trade-off e limitazioni

| Aspetto | Pro | Contro |
|---------|-----|--------|
| NAT Gateway (managed) | Zero manutenzione, autoscaling | Costo maggiore, meno flessibile |
| NAT Instance (unmanaged) | Costo inferiore, usabile come bastion/VPN | Gestione OS/patch a carico tuo |
| Elastic IP | IP fisso e portabile | Billable anche da ferma ($0.005/h) |
| Public IP auto-assign | Gratuito | Cambia ad ogni stop/start |
| Security Group stateful | Non serve regola per la risposta | Nessun DENY esplicito |
| Network ACL stateless | Controllo granulare bidirezionale | Serve configurare entrambe le direzioni |
| Single IGW per VPC | Semplice | Non si possono usare IGW diversi per subnet diverse |
| CIDR fisso | - | Non modificabile dopo la creazione — scegliere /16 fin dall'inizio |

---

## Sicurezza e best practice

1. **Eliminare subito la Default VPC** su ogni nuovo account AWS in ogni regione
2. **Non dare Public IP alle istanze** se non strettamente necessario — usare load balancer/NAT
3. **Separare security controls**: network admin gestisce NACL, dev/ops gestisce Security Groups (defense in depth)
4. **Default route table local-only**: nuove subnet sono automaticamente private
5. **Nascondere servizi unmanaged dietro managed**: metti gli EC2 dietro un Load Balancer managed (ALB/NLB) — lascia che AWS assorba gli attacchi prima che arrivino alle tue istanze
6. **Multi-AZ**: sempre creare subnet in almeno 2 AZ per alta disponibilità
7. **CIDR non sovrapposti**: evitare overlap con on-premise o altri VPC per facilitare il routing

---

## Q&A notevoli

**Q**: Posso creare solo un Internet Gateway per VPC?
**A**: Sì, un VPC può avere al massimo 1 IGW. Non è possibile usare IGW diversi per subnet diverse. Si possono però avere più route table che puntano allo stesso IGW.

**Q**: I subnet pubblici possono usare il NAT Gateway invece dell'IGW?
**A**: Tecnicamente sì, ma non avrebbe senso: il punto di una subnet pubblica è l'accesso diretto a internet. Usare NAT in una public subnet elide il vantaggio.

**Q**: Se un'istanza privata usa la route table con IGW, funziona?
**A**: No. L'IGW manda traffico su internet usando il Public IP, ma l'istanza privata non ha un Public IP. L'internet non sa come rispondere a un IP privato.

**Q**: Le regole di Security Group sono definite a livello EC2?
**A**: No. Il Security Group è un oggetto separato applicabile a più istanze. Modificare il SG aggiorna automaticamente tutte le istanze associate.

**Q**: Un Elastic IP è sempre billable anche se non associato?
**A**: Sì. $0.005/ora indipendentemente dall'utilizzo.

**Q**: La Network ACL è collegata alla Route Table?
**A**: No. Entrambe sono associate a subnet, ma in modo indipendente: la route table governa il routing, la NACL governa il filtro del traffico.

**Q**: Un'istanza con public IP è reachable da internet anche senza regole nel Security Group?
**A**: No. Il Security Group blocca il traffico inbound non permesso anche se l'istanza ha un Public IP visibile. Il Public IP rende l'istanza *indirizzabile* ma non *accessibile* senza regole esplicite.

**Q**: Inbound/outbound sono relativi al Public IP?
**A**: No. Sono relativi all'istanza EC2. Anche con public IP, le regole di SG e NACL controllano cosa entra e cosa esce. Avere un public IP senza regole non crea vulnerabilità di per sé — ma è un rischio potenziale se si commettono errori di configurazione (meglio non avere il public IP se non serve).

---

## 📚 Risorse e riferimenti

### Corso companion (video on-demand)
- **AWS VPC Hands-on** — Video course di Rick Crisci su O'Reilly — approfondisce ulteriormente le demo

### Corso gratuito
- **IP Addressing and Subnetting Basics** — Rick Crisci — corso gratuito (coupon condiviso durante la sessione) — copre: public/private IP, classful vs classless, CIDR notation, broadcast address, default gateway

### AWS Documentation
- RFC1918 private IP ranges (10.x.x.x, 172.16-31.x.x, 192.168.x.x)
- AWS VPC Pricing page — prezzi Elastic IP, NAT Gateway, PrivateLink

### Certificazioni correlate
- **AWS Solutions Architect Associate (SAA-C03)** — argomenti VPC: CIDR /16 max, public subnet = route to IGW, main route table, stateful/stateless
- **AWS Cloud Ops Engineer** — nuovo esame che sostituisce SysOps SOA-C02 (in arrivo fine 2025)
- **AWS Networking Specialty** — approfondimento networking avanzato

### Tool e servizi AWS citati
- **Amazon VPC** — Virtual Private Cloud
- **EC2 (Elastic Compute Cloud)** — virtual machine managed
- **Internet Gateway (IGW)** — connessione VPC → internet
- **NAT Gateway** — internet outbound per subnet private (managed)
- **Elastic IP** — IP pubblico statico
- **ENI (Elastic Network Interface)** — interfaccia di rete portabile
- **Security Groups** — firewall stateful a livello di interfaccia
- **Network ACL** — firewall stateless a livello di subnet
- **Route Tables** — routing interno alla VPC
- **"VPC and more" wizard** — creazione automatica VPC completa
- **AWS Shield** — protezione DDoS (trattato giorno 2)
- **ALB/NLB (Application/Network Load Balancer)** — managed layer davanti agli EC2 (trattato giorno 2)
- **PrivateLink** — connessione privata tra VPC senza peering (trattato giorno 2)

### Argomenti del Giorno 2
- VPC Peering (come connettere VPC diversi)
- PrivateLink
- Load Balancers (ALB/NLB) + sicurezza
- VPN e Direct Connect
- AWS Shield e WAF
- Pricing dettagliato NAT Gateway, PrivateLink
- Multi-account strategy

---

## Takeaway applicabili

**Giorno 1:**
1. **Eliminare Default VPC** — effort: basso (5 min) — applicare su ogni nuovo account AWS in ogni regione
2. **Struttura VPC standard** — effort: medio (1h) — creare VPC con subnet pubbliche/private in 2+ AZ, IGW, NAT GW, route tables separate — è lo scheletro di qualsiasi workload AWS
3. **Security Group granulari** — effort: basso — creare SG per ruolo (web-servers, databases, bastion) invece di SG unici per tutto
4. **Evitare Public IP diretti** — effort: medio — mettere EC2 dietro ALB con solo private IP sulle istanze
5. **CIDR /16 fin dall'inizio** — effort: zero (scelta al momento della creazione) — non modificabile dopo

**Giorno 2:**
6. **VPC Gateway Endpoint per S3** — effort: bassissimo (5 min) — attivare su ogni VPC che usa S3: traffico rimane su backbone, zero costo aggiuntivo
7. **VPC Flow Logs abilitati** — effort: basso — attivare su ogni VPC di produzione su S3: visibilità su traffico accettato/rifiutato per troubleshooting e compliance
8. **Load Balancer davanti agli EC2 web** — effort: medio — ALB/NLB in subnet pubblica, istanze in subnet private: sicurezza + HA automatica
9. **IaC per la VPC** — effort: medio — definire la VPC via CloudFormation o Terraform: rollback rapido, dev/prod parity, deploy multi-regione in minuti
10. **Non sovrapporre CIDR tra VPC** — effort: zero (design time) — CIDR non overlapping ora risparmia ore di debug se in futuro serve VPC peering

---

## Materiali aggiuntivi disponibili

- [x] Trascrizione audio Giorno 1: `/tmp/course_transcript.txt` (148KB, 2034 righe)
- [x] Trascrizione audio Giorno 2: `/tmp/course_transcript_day2.txt` (153KB, 2816 righe)
- [x] Slide PDF ufficiali: `awsvpcs1262311753921450711.pdf` (32 slide — Pearson/O'Reilly) — integrate in iterazione 3
- [ ] Companion video course on-demand su O'Reilly

---

## Domande di esame (Poll Slides ufficiali)

Queste domande compaiono nelle slide ufficiali del corso e rispecchiano il formato dell'esame AWS.

| Domanda | Risposta corretta |
|---|---|
| Qual è lo scope di una VPC? | **Regional** |
| Qual è lo scope di una subnet? | **Availability Zone** |
| Qual è il CIDR range massimo di una VPC? | **/16** |
| Quale route table viene usata per le nuove subnet? | **Main route table** |
| Cos'è una public subnet? | **Una subnet con una route verso l'Internet Gateway** |
| Caratteristiche di un Elastic IP? (scegli 2) | **Può essere staccato e riattaccato a un'altra istanza** + **È un indirizzo IP pubblico** |
| Che tipo di firewall è un Network ACL? | **Stateless** |
| Passi necessari per usare VPC Peering? (scegli 2) | **Le Route Table devono essere configurate** + **La Peering Connection deve essere accettata** |
| Cosa può connettere un Transit Gateway? | **Tutto: VPC, VPN, DX Gateway, altri Transit Gateway** |
| Come aiuta l'ELB con la disponibilità? (scegli 2) | **Distribuisce traffico tra istanze in più AZ** + **Esegue health check periodici** |

> **Nota esame**: l'Elastic IP è billable sempre — anche quando NON è associato a un'istanza in esecuzione. La slide dice "You are only billed when attached to a running instance" ma nella pratica AWS fattura anche se l'EIP è allocato ma non associato a nulla.

---

# GIORNO 2 — Argomenti Avanzati

---

## Concetti fondamentali — Giorno 2

### VPC Peering

Una connessione di peering VPC crea un **link diretto di routing tra due VPC** che permette alle istanze di comunicare come se fossero sulla stessa rete. Il traffico non attraversa mai internet: resta sull'**AWS backbone**.

**Caratteristiche:**
- Funziona tra VPC nello stesso account, account diversi, stessa regione, regioni diverse
- Limite soft: max **50 peering connection per VPC**, aumentabile fino a **125 max** (confermato nelle slide)
- Per cross-account: una VPC invia la richiesta, l'altra deve accettarla
- Non esiste il **transitive peering**: non si può mandare traffico da VPC-A → VPC-B → VPC-C. Serve una connessione diretta A↔C
- No hardware speciale, nessun single point of failure (managed)

**Requisito critico — nessun overlap di CIDR**: se due VPC hanno range sovrapposti, le route table entries sono impossibili da configurare (la local route copre già quell'indirizzo).

**Configurazione necessaria:**
1. Creare la peering connection
2. Accettare la richiesta (richiede l'altra parte se cross-account)
3. Aggiornare le **route table di entrambe le VPC** per includere il CIDR della VPC remota via peering connection

**VPC Peering vs VPN fai-da-te:**
| | VPC Peering | VPN tra EC2 |
|---|---|---|
| Gestione | AWS managed | Tu gestisci tutto |
| Traffico | AWS backbone | Internet (o non) |
| Disponibilità | Alta (AWS garantisce) | Dipende dalla tua istanza EC2 |

**Con Service Mesh (es. Istio):** VPC Peering fornisce la connettività di rete (layer 3/4). Il service mesh (Istio, Linkerd) siede sopra e aggiunge service discovery, traffic shaping, mTLS tra i singoli servizi. Sono soluzioni complementari.

### Transit Gateway

Il Transit Gateway è un **hub centrale** che risolve il problema del full-mesh con molte VPC. Invece di N×(N-1)/2 peering connections, ogni VPC si connette al Transit Gateway.

```
VPC-A ─┐
VPC-B ─┤
VPC-C ─┤── Transit Gateway ──── On-prem / VPN / Direct Connect
VPC-D ─┤
VPC-E ─┘
```

**Vantaggi:**
- Una sola connessione per VPC (semplifica il management)
- Può anche aggregare connessioni on-premise (VPN o Direct Connect)
- Può connettersi ad altri Transit Gateway (multi-regione, multi-account)
- AWS managed: alta disponibilità inclusa (opzione multi-AZ)

**Cosa può connettere un Transit Gateway (risposta esame: tutti):**
- VPC
- VPN (Customer Gateway on-prem)
- Direct Connect Gateway (DXGW)
- Altri Transit Gateways

**Costi:**
- $0.05/ora per ogni attachment (VPC, VPN, Direct Connect collegato)
- $0.02/GB di traffico processato

**Transit VPC (approccio vecchio):** deploy di un router di terze parti (es. Cisco, Palo Alto) dentro una VPC normale per fare da hub. Ancora usato quando:
- Non si vuole pagare il Transit Gateway
- Serve deep packet inspection layer 7 con NGFW (Fortinet, Palo Alto) su tutto il traffico inter-VPC

**Esame tip**: Transit Gateway → "ho molte VPC che devono comunicare". VPC Peering → "ho 2 VPC da connettere".

### Elastic Load Balancer (ELB)

**Motivazione sicurezza**: invece di esporre le istanze EC2 con Public IP direttamente su internet, si mette un load balancer davanti. Gli attaccanti colpiscono il servizio managed di AWS, non le nostre istanze.

**Motivazione disponibilità**: il load balancer distribuisce il traffico tra istanze in AZ diverse. Se un'AZ fallisce, smette di mandare traffico a quelle istanze (health checks).

**Tipi:**
| Tipo | Layer | Caso d'uso |
|---|---|---|
| Network Load Balancer (NLB) | 4 (TCP/UDP) | Alta performance, IP fisso, protocolli non-HTTP |
| Application Load Balancer (ALB) | 7 (HTTP/HTTPS) | Content-based routing (path, host), più applicazioni sullo stesso LB |

**Due tipi di Load Balancer per posizionamento:**

**Internet-Facing ELB** (dal diagramma slide):
```
Route 53 (www.example.com)
    ↓
Internet-Facing ELB  ──→  Web Server (AZ1)
                     ──→  Web Server (AZ1)
                     ──→  Web Server (AZ2)
                     ──→  Web Server (AZ2)
```
- Ha un DNS pubblico puntato da Route 53
- Distribuisce il traffico su istanze in più AZ

**Internal ELB** (dal diagramma slide — uso tier-to-tier):
```
Route 53 → Web Server instance
                  ↓
           Internal ELB  ──→  App Server (no public IP)
                         ──→  App Server
                         ──→  App Server
                         ──→  App Server
```
- Usato per comunicazione tra tier (web tier → app tier)
- NON esposto su internet — solo accessibile internamente alla VPC
- Permette scaling indipendente del tier app senza cambiare la configurazione del web tier

**Target Group**: gruppo di target (EC2, IP, Lambda) a cui il LB distribuisce traffico. Può essere un Auto Scaling Group per scaling automatico.

**Health checks**: il LB interroga periodicamente ogni istanza. Se una non risponde → rimossa dalla rotation finché non torna healthy.

**DNS invece di IP**: il LB ha un DNS name pubblico (non un IP) perché internamente è distribuito su più AZ — gli IP possono cambiare.

**Scope**: regionale, distribuito sulle AZ selezionate alla creazione.

### VPC Endpoints

Permettono alle risorse dentro una VPC di comunicare con servizi AWS senza che il traffico attraversi internet.

**Gateway Endpoint (gratuito):**
- Solo per **S3 e DynamoDB**
- Modifica automaticamente la route table aggiungendo una entry con prefix list
- Il prefix list contiene tutti gli IP di S3 (o DynamoDB) in quella regione
- Gestito da AWS (aggiornato automaticamente)
- Creato di default dal wizard "VPC and more"

**Prefix list**: struttura gestita da AWS che contiene tutti gli IP di un servizio in una regione. La route table punta al gateway endpoint se la destinazione è in quel prefix list.

**Interface Endpoint (billable):**
- Per tutti gli altri servizi AWS (280+: EC2 API, SSM, CloudWatch, ecc.)
- Crea un'ENI nella subnet con IP privato
- Non richiede modifiche alla route table

**PrivateLink**: il meccanismo che abilita gli interface endpoints. Permette anche di **esporre un proprio servizio** ad altri account tramite VPC endpoint. Esempio:
- Azienda A ha un servizio (avatar generator) dietro un NLB
- Azienda B si connette al servizio di A tramite interface endpoint
- Il traffico non esce mai su internet

**Chiarimento S3**: l'S3 bucket è nello stesso account e stessa regione della VPC, ma non è **dentro** la VPC. Senza VPC endpoint, il traffico EC2→S3 passa per internet. Con VPC endpoint, passa sull'AWS backbone.

**Multi-account per isolamento massimo**: separare team in AWS account diversi (HR, Finance, Customer Service). Gestione centralizzata tramite **AWS Organizations**. Più sicuro che affidarsi solo a policy IAM all'interno dello stesso account.

### VPC Flow Logs

Registrano le **informazioni sul traffico IP** che transita attraverso le interfacce di rete nella VPC. Analogo a NetFlow/IPFIX nel networking tradizionale.

**Configurazione:**
- Livello: VPC, subnet, o singola ENI
- Filtro: traffico accettato, rifiutato, o entrambi
- Destinazione: S3 bucket o CloudWatch Logs

**Formato log:** interfaccia di rete sorgente, IP sorgente, IP destinazione, porta, protocollo, bytes, stato (ACCEPT/REJECT).

**Uso tipico:**
- Troubleshooting: capire perché il traffico non arriva (security group? NACL?)
- Security: rilevare traffico anomalo
- Compliance: audit trail del traffico

**Esame SysOps**: bisogna saper leggere i flow log e interpretare perché il traffico è stato accettato o rifiutato.

### Direct Connect

Connessione **fisica dedicata** dal proprio data center ad AWS. Non passa per internet — traffico sull'AWS backbone.

**Infrastruttura necessaria:**
```
Data Center On-Prem
      │ (connessione fisica acquistata da ISP: 1G/10G/100G)
      ▼
Colocation Facility (Direct Connect Location)
   ├── Il tuo router (da te installato e gestito)
   └── Router AWS (si connette all'AWS backbone)
            │ (cross-connect richiesto a AWS)
            ▼
       AWS Backbone → Virtual Private Gateway (VGW) → VPC
```

**Passi per attivarlo:**
1. Acquistare connessione fisica (1G, 10G, o 100G) dal proprio data center al Direct Connect Location
2. Installare un proprio router nel colocation facility
3. Richiedere ad AWS il cross-connect per collegare il proprio router all'AWS router
4. Configurare Virtual Private Gateway (VGW) nel VPC
5. Configurare routing (BGP)

**Benefici chiave (per l'esame):**
- **Throughput predicibile e affidabile** (non varia come internet)
- **Latenza bassa e consistente**
- **Costo data transfer ridotto**: $0.02/GB vs $0.09/GB del VPN
- Riduce l'uso di internet — utile per grandi volumi di dati

**VPN over Direct Connect**: si può aggiungere un layer di crittografia IPsec sopra la connessione Direct Connect (gratuito). Molte organizzazioni lo fanno perché il traffico transita in facility di terze parti non controllate.

**Virtual Interface (VIF)** — concetto dalle slide, non coperto nella trascrizione:
Il VIF è la **connessione logica** che viaggia sopra il circuito fisico Direct Connect. Configurazione richiesta:
- VGW (lato AWS)
- AWS Account
- VLAN
- Auto-generate IPs o IP manuali
- Customer Router Peer IP (lato on-prem)
- Amazon Router Peer IP (lato AWS)
- BGP ASN

```
AWS Account/VPC
      │
     VGW
      │
[AWS Hw. Router] ←── VIF Connection ──→ [Customer Router] ──WAN──→ [Router BGP] → On Prem subnets
```

**Direct Connect Gateway (DXGW)** — concetto dalle slide, non coperto nella trascrizione:
Il DXGW è un hub che permette a **una singola connessione Direct Connect** di raggiungere **più VPC** (anche in regioni diverse o account diversi).

```
On Prem → WAN → Customer Router → DX Router (colocation)
                                        ↓
                                      DXGW
                                    ↙ ↙ ↙ ↙ ↙
                             VPC1 VPC2 VPC3 VPC4 VPC5
```

Senza DXGW: ogni VPC richiederebbe un VGW dedicato e una connessione separata.
Con DXGW: un solo circuito fisico, un solo DX Router, raggiunge tutte le VPC collegate.

### Managed Hardware VPN

Alternativa al Direct Connect: VPN IPsec **sopra internet**, gestita da AWS.

**Componenti:**
- **Customer Gateway (CGW)**: il tuo router on-premise — AWS ne ha bisogno per configurare la VPN
- **Virtual Private Gateway (VGW)**: lato AWS — diverso dall'IGW, solo per VPN
- **CPE (Customer Premises Equipment)**: il dispositivo fisico on-prem dove termina il tunnel VPN (ha un Public IP per IPsec)

**Configurazione VPN statico** (dalle slide):
```
VPC (AZ1/AZ2)
    ↓
   CGW → VGW ─────(IPsec tunnel over internet)─────→ CPE (Public IP) → On Prem
```
Parametri necessari:
- Name
- VGW e CGW da collegare
- Tipo: Static (routing statico) o Dynamic (BGP)
- IKE Pre-shared Key (per autenticazione tunnel)

**Come si differenzia dall'IGW:**
- IGW: per qualsiasi traffico internet
- VGW: esclusivamente per VPN connections

**Caratteristiche:**
- Facile da configurare (solo internet access necessario)
- IPsec — tutto il traffico è cifrato
- Nessun hardware fisico richiesto
- Costo egress: $0.09/GB

**Software VPN**: alternativa dove si installa il software VPN su un'istanza EC2. Utile quando le normative impongono di gestire direttamente l'hardware VPN (VGW non è un'opzione).

**Confronto Direct Connect vs Managed VPN:**
| Aspetto | Managed VPN | Direct Connect |
|---|---|---|
| Setup | Ore | Settimane/mesi (circuito fisico) |
| Throughput | Variabile (internet) | Predicibile, dedicato |
| Latenza | Variabile | Bassa e costante |
| Transfer out | $0.09/GB | $0.02/GB |
| Encryption | IPsec incluso | Opzionale (gratuito se aggiunto) |
| Costo fisso | Nessuno | Port fee + ISP |

**Quando scegliere Direct Connect:** grandi volumi di trasferimento dati dove il risparmio sul transfer ($0.07/GB × volume) supera il costo del circuito fisico.

### Data Transfer Charges

Le spese di trasferimento "si infilano" nel conto se non si conoscono le regole.

**Regole fondamentali:**

| Tipo traffico | Costo |
|---|---|
| Ingress verso AWS | Gratuito |
| Egress internet (da EC2) | $0.05–$0.09/GB |
| Egress via Direct Connect | $0.02/GB |
| Egress via VPN | $0.09/GB |
| Tra AZ diverse (stessa regione) | $0.01/GB |
| All'interno della stessa AZ | Gratuito |
| Cross-region (qualsiasi metodo) | Sempre a pagamento |
| Tramite NAT Gateway | A pagamento |
| Tramite VPC Endpoint (S3, DynamoDB nella stessa regione) | Gratuito |
| S3, DynamoDB, Kinesis, SQS — stessa regione, via endpoint | Gratuito |

**Trappole comuni:**
- EC2 in AZ-a comunica con RDS in AZ-b → paga $0.01/GB
- Pensare che "stessa regione = gratuito" — sbagliato: cambiare AZ è billable
- NAT Gateway aggiunge costi al traffico che passa attraverso

**Riferimento**: AWS pubblica il "Data Transfer Charges Diagram" (versione 2021 ancora utile per capire la struttura, anche se i prezzi sono aggiornati).

### Infrastructure as Code (IaC)

Gestire l'infrastruttura tramite template versionati invece di click manuali nella console.

**CloudFormation (AWS native):**
- Template YAML o JSON
- "Stack" = gruppo di risorse create dal template
- Versioning: ogni template è una versione, rollback immediato a versione precedente
- Uso pratico: dev e prod usano lo stesso template → ambienti identici

**Terraform (preferito da Rick):**
- Multi-cloud (AWS, GCP, Azure, on-prem)
- Stesso apprendimento, applicabile ovunque
- Più diffuso di CloudFormation per team che usano più cloud
- Rick lo preferisce a CloudFormation nonostante insegni AWS

**Workflow con AI (mostrato in demo):**
1. Chiedere a Perplexity (o altro LLM): "Write a CloudFormation template to create a VPC in us-west-1 with 2 public and 2 private subnets across AZs, internet gateway for public subnets, NAT gateway for private subnets"
2. Il tool genera il template YAML completo e corretto
3. Salvare come `.yaml` e caricare in CloudFormation
4. Testare → iterare → versionare

**Valore del versioning infrastruttura:**
- Recupero rapido da config errata (rollback alla versione precedente)
- Parity dev/prod: stesso template, parametri diversi
- Deploy in regioni diverse con lo stesso template

**Accesso programmatico per Terraform/CLI:**
- IAM → Utente → Security Credentials → Access Keys
- Le access key permettono chiamate API programmatiche (Terraform, AWS CLI, SDK)

---

## Demo e esempi pratici — Giorno 2

### Demo 16 — Crea VPC Peering
Creare una seconda VPC ("peer VPC", CIDR 10.2.0.0/16) con "VPC and more". Navigare su VPC → Peering Connections → Create → chiamare "rick-demo-peer" → scegliere VPC locale (rick-demo) e VPC peer. Poiché stesso account: accettare subito la request.

**Step cruciali dopo la creazione:**
1. Editare route table di rick-demo: aggiungere `10.2.1.0/24 → peering connection`
2. Editare route table di peer-VPC: aggiungere `10.1.1.0/24 → peering connection`

**Verifica:** creare EC2 in peer-VPC subnet pubblica → ping 10.2.1.x da istanza in rick-demo → funziona. Traceroute: 0 hop intermedi (traffico su AWS backbone, non internet).

**Latenza misurata**: 0.1–0.2ms — alta velocità, bassa latenza.

### Demo 17 — Load Balancer con 4 web server
1. Creare 3 istanze EC2 aggiuntive (totale 4 web server)
2. EC2 → Load Balancers → Create → Network Load Balancer
   - Nome: "demo-NLB"
   - Internet-facing
   - VPC: rick-demo
   - Subnets: scegliere più AZ
   - Security group: web-servers
3. Creare Target Group "demo-TG" → tipo: istanze EC2 → porta 80
4. Aggiungere le 4 istanze come target
5. Listener: TCP:80 → forward a demo-TG
6. Aspettare health checks "healthy" (qualche minuto)
7. Copiare il DNS name del LB → incollare nel browser → sito risponde

**Risultato**: le 4 istanze sono in rotation. Se 3 falliscono, il sito continua a funzionare.

### Demo 18 — VPC Endpoint per S3
1. VPC → Endpoints → Create → nome "S3-VPC-E"
2. Tipo: Gateway → servizio: S3 (com.amazonaws.us-west-1.s3)
3. Selezionare VPC e tutte le route table su cui aggiornare automaticamente
4. Create

**Verifica:** Route Tables → internet-access → ora appare una terza route:
```
Destination: pl-xxxxxx (S3 prefix list)   Target: vpce-xxxxxx
```
Da questo momento tutto il traffico S3 dalla VPC va sull'AWS backbone, non su internet.

### Demo 19 — Virtual Private Gateway e Managed VPN
1. VPC → Virtual Private Gateways → Create "VGW-demo"
2. Attach alla VPC
3. VPC → Customer Gateways → Create → nome "my-on-prem-router" → inserire IP del router on-prem
4. VPC → VPN Connections → Create "demo-VPN" → VGW + CGW → configurare CIDR on-prem (192.168.0.0/16) e CIDR AWS (10.1.1.0/24) → tipo: IPsec

### Demo 20 — CloudFormation con AI
1. Aprire Perplexity (o altro LLM)
2. Prompt: "Write a CloudFormation template to create a VPC in us-west-1, 2 public and 2 private subnets across AZs, internet gateway for public subnets, NAT gateway for private subnets"
3. Copiare YAML generato → salvare come `testvpc.yaml`
4. CloudFormation → Create Stack → Upload template → nome "demo" → Submit
5. Il template crea automaticamente: VPC, subnets, IGW, route tables, NAT gateway, associazioni

**Risultato**: stessa infrastruttura del corso creata in 2 minuti invece di 1 ora di click manuali.

---

## Trade-off e limitazioni — aggiornamento Giorno 2

| Aspetto | Pro | Contro |
|---------|-----|--------|
| VPC Peering | Semplice, managed, traffico su backbone | No transitive peering, full-mesh con molte VPC |
| Transit Gateway | Hub centrale, semplifica N VPC | Costo ($0.05/h per attachment + $0.02/GB) |
| Transit VPC | Layer 7 inspection con NGFW, costo flessibile | Gestione router a tuo carico, single point of failure se non HA |
| NLB | Alte performance, layer 4 | Meno funzionalità routing di ALB |
| ALB | Content-based routing, più app per LB | Slightly higher cost |
| Gateway VPC Endpoint | Gratuito, no overhead | Solo S3 e DynamoDB |
| Interface VPC Endpoint | 280+ servizi, PrivateLink | Billable (ENI + ore) |
| Direct Connect | Throughput predicibile, $0.02/GB egress | Settimane/mesi setup, costo fisso circuito |
| Managed VPN | Setup rapido, nessun hardware fisico | $0.09/GB, performance dipende da internet |
| Software VPN | Controllo totale, compliance mandatoria | Gestione OS/HA a carico tuo |
| CloudFormation | Native AWS, gratuito | Solo AWS, meno ecosistema di Terraform |
| Terraform | Multi-cloud, ecosistema vasto | Curva apprendimento, non managed da AWS |

---

## Q&A notevoli — Giorno 2

**Q**: Perché non usare un VPN invece del VPC peering?
**A**: Si può, ma il VPC peering è managed (AWS lo gestisce), il traffico va sull'AWS backbone (non su internet), e ha alta disponibilità built-in. Un VPN fai-da-te su EC2 richiede gestione e fallisce se l'istanza cade.

**Q**: Esiste il transitive VPC peering?
**A**: No. AWS non lo supporta. Non si può passare da VPC-A a VPC-B a VPC-C tramite peering. Per connettere N VPC usare Transit Gateway.

**Q**: Per il cross-account peering servono credenziali speciali?
**A**: No. L'account richiedente conosce l'account number dell'altro. L'altro account deve accettare la peering request. Nessuna credenziale aggiuntiva.

**Q**: Il load balancer può stare davanti all'internet gateway?
**A**: No — non è controllabile dall'utente. Il LB è un servizio managed AWS e si colloca "dopo" l'IGW nell'architettura.

**Q**: S3 è dentro la VPC?
**A**: No. S3 esiste nello stesso AWS account e nella stessa regione, ma è fuori dalla VPC. Il traffico EC2→S3 passa per internet per default. Con un VPC Gateway Endpoint rimane sull'AWS backbone.

**Q**: Il traffico nella stessa regione tra AZ è gratuito?
**A**: No. Traffico tra AZ diverse nella stessa regione è $0.01/GB. Solo il traffico all'interno della stessa AZ è gratuito.

**Q**: VPN over Direct Connect è a pagamento?
**A**: No — aggiungere crittografia IPsec sopra una connessione Direct Connect è gratuito.

**Q**: Per usare Terraform con AWS servono credenziali speciali?
**A**: Sì. IAM → Utente → Security Credentials → Create Access Key. Le access key permettono le chiamate API programmatiche. Tenere le access key segrete (non committatele in git).

**Q**: Posso fare inferencing di Claude o Perplexity su EC2?
**A**: Claude e Perplexity sono accessibili solo via API (Bedrock per Claude). Non si può fare girare Claude direttamente su un'istanza EC2. Si può invece scaricare e girare modelli open source su EC2 con GPU.

---

## 📚 Risorse e riferimenti — aggiornamento Giorno 2

### Tool e servizi AWS aggiuntivi (Giorno 2)
- **VPC Peering** — connessione diretta tra 2 VPC sullo stesso backbone AWS
- **Transit Gateway** — hub per connettere molte VPC e on-prem
- **Virtual Private Gateway (VGW)** — lato AWS per VPN e Direct Connect
- **Customer Gateway (CGW)** — rappresentazione del router on-prem in AWS
- **Elastic Load Balancer (ELB)** — family: ALB, NLB, Gateway LB
- **VPC Endpoints** — Gateway (S3/DynamoDB free) e Interface (280+ servizi billable)
- **PrivateLink** — meccanismo per esporre servizi ad altri account via endpoint
- **VPC Flow Logs** — cattura metadati traffico di rete (come NetFlow)
- **Direct Connect** — circuito fisico dedicato 1G/10G/100G verso AWS backbone
- **AWS CloudFormation** — IaC nativa AWS (YAML/JSON templates)
- **AWS Organizations** — gestione centralizzata di più account AWS
- **AWS Bedrock** — modelli AI managed su AWS (Claude, Titan, ecc.)

### Tool esterni citati
- **Terraform** — IaC multi-cloud, preferito da Rick rispetto a CloudFormation
- **Perplexity** — AI tool di riferimento di Rick per generare template e ricerche
- **Cisco, Fortinet, Palo Alto** — vendor di NGFW usabili come Transit VPC router
- **AWS Distributed Load Testing** — soluzione AWS per load testing (white paper)
- **Miro** — whiteboard usata per i diagrammi durante il corso

### Corsi futuri di Rick Crisci
- **Perplexity** — come usare Perplexity e Perplexity Labs per progetti complessi
- **AI for Business Tasks** — risparmio tempo con AI
- **AWS Certified Cloud Ops Engineer** — previsto gennaio (sostituisce SysOps SOA-C02)
- **Containers on AWS (ECS/EKS)** — da rimettere in calendario

### Risorse AWS ufficiali
- **AWS Well-Architected Framework** — Security Pillar PDF: raccomanda servizi managed e riduzione dell'attack surface
- **Data Transfer Charges Diagram** — diagramma AWS (versione 2021) per capire cosa costa il trasferimento dati
- **Direct Connect Service Level Agreement** — garantisce uptime del servizio
- **AWS Service Quotas** — dove richiedere aumenti dei soft limit (es. 50 peering → 125)
