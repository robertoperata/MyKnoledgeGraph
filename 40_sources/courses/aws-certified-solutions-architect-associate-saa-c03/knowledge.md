---
titolo: AWS Certified Solutions Architect Associate (SAA-C03) Crash Course
piattaforma: O'Reilly Live Events
istruttore: Chad Smith
data: 2025-03-26
durata_totale: ~7.7h (entrambe le sessioni trascritte)
trascrizione_coperta: 7.6h su ~7.7h (99%) — Day1 3.9h + Day2 3.73h
lingua_originale: en
iterazione: 2
tags:
  - aws
  - solutions-architect
  - saa-c03
  - cloud
  - certification
feature: 
type: live event
author: Chad Smith
source: 
---

## Chi è l'istruttore

**Chad Smith** è un professionista della tecnologia, autore e trainer focalizzato su infrastruttura cloud e management. Ricopre il ruolo di **Principal Cloud Architect** presso brightkey.cloud.

Certificazioni attuali:
- AWS: Architecture, SysOps, Security, Networking, Databases
- CompTIA Cloud+

Ha una vasta esperienza pratica in cloud adoption, infrastructure design, operations e security. Ha partecipato al programma AWS SME (Subject Matter Expert) per la scrittura delle domande degli esami, ma ha scelto di non completare il processo per evitare un conflitto di interesse con la sua attività di docenza.

---

## A chi è rivolto e obiettivi dichiarati

Il corso è rivolto a chi:
- Si prepara all'esame AWS SAA-C03
- Ha esperienza con AWS ma vuole strutturare le proprie conoscenze
- Vuole comprendere il AWS Well-Architected Framework come fondamento di tutto il lavoro su AWS

**Raccomandazione di Chad sull'ordine delle certificazioni:**
1. AWS Cloud Practitioner (opzionale ma utile per imparare il processo dell'esame)
2. **AWS Solutions Architect Associate** — obbligatoria per tutti, indipendentemente dalla specializzazione (developer, security, networking, operations)
3. Security Specialty (seconda più importante in assoluto)
4. Poi la specializzazione di propria competenza

Motivazione: il Well-Architected Framework è la base di letteralmente tutto in AWS, non solo dell'architettura.

---

## Struttura del corso

Questa sessione (Giorno 1 di 2) copre:

1. **Introduzione all'esame SAA-C03** — struttura, domande, strategie
2. **AWS Well-Architected Framework** — tutti e sei i pilastri
3. **Question Domain 1: Security** — tutti i principi del pillar Security
4. **Question Domain 2: Reliability** — principi del pillar Reliability (iniziato, continua nel giorno 2)

**Giorno 2** (trascritto nella seconda iterazione):
4. **Question Domain 2: Reliability** — completamento (Stop Guessing Capacity, Manage Change through Automation, Deployment Types, Immutable Infrastructure)
5. **Question Domain 3: Performance Efficiency** — completo
6. **Question Domain 4: Cost Optimization** — completo (Cloud Financial Management, EC2 pricing, CUR)

---

## Struttura dell'esame SAA-C03

### Dati numerici fondamentali

| Parametro | Valore |
|-----------|--------|
| Numero di domande | 65 |
| Tempo disponibile | 130 minuti |
| Domande non valutate | 15 (usate per testare nuove domande) |
| Penalità per risposta errata | Nessuna |
| Tipologia | Multiple choice + Multiple response |

🎯 **Importante per l'esame:** Le domande a risposta multipla (2-3 risposte corrette) non danno punteggio parziale — bisogna indovinare TUTTE le risposte corrette. Ogni domanda ha un punteggio variabile in base alla difficoltà.

### Quattro question domain (pillar in scope)

| Domain | Pillar | Percentuale |
|--------|--------|-------------|
| 1 | Security | 30% |
| 2 | Reliability | ~26% |
| 3 | Performance Efficiency | ~24% |
| 4 | Cost Optimization | ~20% |

🎯 **Strategie per l'esame:**
- Ogni parola nella domanda e nelle risposte conta
- La skill più importante: **eliminare le risposte errate**
- Flaggare le domande incerte e tornarci dopo
- I servizi "core" (EC2, S3, RDS, IAM, VPC) sono quelli su cui concentrare lo studio
- I servizi minori appaiono in 1-2 domande al massimo
- Non ci sono questioni che mescolano domain diversi (es. security vs. performance come trade-off) — solo al livello Professional

### Pillar NON in scope per SAA-C03
- Operational Excellence
- Sustainability

(Anche se i concetti di questi pillar sono comunque utili per capire il materiale)

---

## AWS Well-Architected Framework — I sei pilastri

### Definizione secondo Chad Smith

> Il Well-Architected Framework insegna come tradurre requisiti di business e tecnici in architettura e operations, seguendo le best practice.

Non è solo architettura: include anche operations e cost management.

---

### Pilastro 1: Operational Excellence

**Definizione:** Supportare lo sviluppo, eseguire i workload efficacemente, ottenere insight dalle operations, migliorare continuamente — **per fornire valore di business**.

Le ultime quattro parole ("to deliver business value") sono fondamentali. Ogni operazione deve poter essere ricondotta a valore di business, altrimenti è difficile giustificarla.

**Sei principi:**
1. Eseguire le operations come codice (IaC: AWS CLI, SDK, CloudFormation, CDK)
2. Frequent small reversible changes (richiede infrastruttura progettata da zero per questo)
3. Anticipate failure
4. Learn from all operational failures
5. Use managed services
6. (altri principi generici)

---

### Pilastro 2: Performance Efficiency

**Definizione:** Usare le risorse di compute in modo efficiente per soddisfare i requisiti, e mantenere tale efficienza al variare della domanda o al cambio di tecnologia.

**Cinque principi, due di interesse speciale:**

- **Democratize advanced technologies:** AWS diventa il subject matter expert per una tecnologia complessa e la offre ai clienti come managed service (es. machine learning, IoT, databases specializzati)

- **Mechanical sympathy:** Scegliere il servizio o la feature che corrisponde esattamente ai requisiti, senza compromessi. Le due aree più difficili dove esibire mechanical sympathy sono **storage** e **databases** — spesso si usa ciò che si conosce meglio, non ciò che è più adatto.

**Risorsa utile:** AWS Decision Guides (pagina ufficiale AWS) — guide molto dettagliate per scegliere tra servizi dello stesso dominio (database, storage, networking). Meno del 20% dei professionisti AWS la conosce.
https://docs.aws.amazon.com/databases-on-aws-how-to-choose/
https://docs.aws.amazon.com/decision-guides/latest/networking-on-aws-how-to-choose/choosing-networking-and-content-delivery-service.html?did=wp_card&trk=wp_card
https://docs.aws.amazon.com/decision-guides/latest/identity-on-aws-how-to-choose/identity-on-aws-how-to-choose.html
https://docs.aws.amazon.com/decision-guides/latest/storage-on-aws-how-to-choose/choosing-aws-storage-service.html
https://docs.aws.amazon.com/decision-guides/latest/sns-or-sqs-or-eventbridge/sns-or-sqs-or-eventbridge.html
https://docs.aws.amazon.com/decision-guides/latest/security-on-aws-how-to-choose/choosing-aws-security-services.html

---

### Pilastro 3: Security

**Definizione:** Proteggere dati e workload secondo i requisiti applicabili.

Il più grande numero di principi tra tutti i pilastri. È il 30% dell'esame.

**Sette principi:**
1. Implement a strong identity foundation
2. Enable traceability
3. Apply security at all layers
4. Automate security best practices
5. Protect data in transit and at rest
6. Keep people away from data
7. Prepare for security events

---

### Pilastro 4: Reliability

**Definizione:** Assicurarsi che i workload eseguano le funzioni previste quando richiesto — inclusa la capacità di operare e testare durante il ciclo di vita.

**Cinque principi:**
1. Automatically recover from failure
2. Test recovery procedures
3. Scale horizontally to increase aggregate system availability
4. Stop guessing capacity
5. Manage change through automation

---

### Pilastro 5: Cost Optimization

**Definizione:** Eseguire i sistemi per fornire valore di business al minor costo possibile.

🎯 **Insight cruciale di Chad:** La cost optimization non è "trovare risorse idle nel tuo account AWS". Quello significa che hai già sbagliato qualcosa. La cost optimization è una **strategia aziendale completa**. Senza visibilità e attribuzione dei costi, si finisce in un loop eterno di "il bill AWS è troppo alto" ogni mese.

**Cinque principi (tutti strategici, nessuno tattico):**
1. Adopt a consumption model
2. Measure overall efficiency
3. Stop spending money on undifferentiated heavy lifting
4. Analyze and attribute expenditure
5. Use managed and application level services

**Nota sull'on-premise repatriation:** Chad è critico verso le aziende che riportano workload on-prem per ridurre i costi. Argomenta che queste analisi considerano solo il costo diretto AWS e trascurano:
- Perdita di agilità
- Costo del personale per gestire hardware
- Tempo sottratto all'innovazione
- La repatriation funziona solo a scala Netflix/Meta

---

### Pilastro 6: Sustainability

**Non è in scope per nessun esame AWS corrente.**

Focalizzato sulla riduzione dell'impatto ambientale. Principale raccomandazione pratica: usare processori **AWS Graviton** (ARM) invece di Intel dove possibile — consumano meno energia, costano meno e spesso hanno performance migliori.

Limitazione: Graviton non supporta Windows (solo workload Linux/containerizzati).

---

## Concetti Fondamentali — Servizi e Feature AWS

### IAM — Identity and Access Management

#### Autenticazione in AWS

Tre modi principali di interagire con AWS:

| Tipo di identità | Identità | Metodo di accesso |
|------------------|----------|-------------------|
| Statica | IAM User | Console (username/password/MFA) o CLI (access key) |
| Temporanea | IAM Role | Assunzione del ruolo via STS |
| Temporanea | Federated User | SAML, OIDC, IAM Identity Center |

Tutti i metodi si basano sugli **API endpoint dei servizi AWS**. È tecnicamente possibile chiamare gli endpoint REST direttamente (es. con curl/Postman) — unico modo per avere il 100% delle funzionalità, ma richiede la generazione manuale delle firme delle richieste.

#### RBAC vs ABAC

| Strategia | Acronimo | Caratteristiche |
|-----------|----------|-----------------|
| Role-Based Access Control | RBAC | Permessi basati sull'identità del principal. Facile da progettare e implementare. Non scala bene nell'auditing. Non è necessariamente least privilege. |
| Attribute-Based Access Control | ABAC | Permessi basati su attributi del principal, della risorsa E della richiesta (IP sorgente, uso TLS, ecc.). Più difficile da progettare, ma scala meglio ed è più vicino al least privilege. |

**Nota terminologica AWS:** AWS usa il termine ABAC per riferirsi specificamente ai **tag** sulle risorse e sui principal. Questa è una definizione riduttiva rispetto al significato originale del termine.

#### Tipi di Policy IAM

**Identity Policies (per chi le riceve il permesso):**
- **AWS Managed** — circa 900 disponibili, read-only, ottimi per iniziare
- **Customer Managed** — create e gestite dal cliente, lifecycle indipendente
- **Inline** — embedded nell'entità IAM (user/group/role), max 2KB (vs 10KB delle managed)

**Resource Policies (contengono l'elemento `Principal`):**
- S3 bucket policy (esempio più comune)
- IAM Role trust policy (determina chi può assumere il ruolo)

**Permission Boundaries:**
- IAM Permission Boundaries — limitano i permessi massimi per IAM user/role
- Organizations SCPs — limitano i permessi per interi account o OU
- Organizations RCPs (Resource Control Policies) — nuovi, annunciati a dicembre 2024, NON ancora in scope per l'esame

**Session Policies:**
Quando si assume un IAM role, si può fornire una policy aggiuntiva che **restringe volontariamente** i permessi a meno di quanto il ruolo concederebbe normalmente. Utile in workflow automatizzati.

**ACL (Access Control List):**
- Formato XML (non JSON)
- Legacy, in via di deprecazione attiva da parte di AWS
- Implementate solo su S3
- Nessuna awareness di IAM
- **Eliminare sempre come risposta corretta** a meno che non siano l'unica soluzione tecnica possibile

#### Ordine di valutazione dei permessi (semplificato)

1. **Explicit Deny** in qualsiasi policy — se presente, accesso negato immediatamente (non superabile nemmeno con permessi admin)
2. **Organizations RCPs** — se abilitati, deve esistere un Allow per l'azione
3. **Organizations SCPs** — come gli RCP, devono contenere Allow
4. **Resource Policy** — se esiste un Allow, si procede
5. **Identity Policy** — deve contenere un Allow
6. **IAM Permission Boundaries** — deve contenere un Allow
7. **Session Policy** (se applicabile) — deve contenere un Allow

🎯 **Per l'esame:** I RCP non sono ancora in scope (annunciati a dicembre 2024 — servono almeno 6 mesi dalla GA).

#### Instance Profile vs IAM Role

Differenza critica spesso nascosta dalla console AWS:

- Non si può associare direttamente un IAM Role a un'istanza EC2
- Esiste un oggetto intermedio chiamato **Instance Profile** che ha il proprio ARN
- La console crea automaticamente l'Instance Profile, quindi l'utente non se ne accorge
- **Con Terraform o CloudFormation è necessario creare esplicitamente l'Instance Profile**

---

### AWS Organizations e Multi-Account Management

#### Organizations

- Struttura gerarchica di account in **Organizational Units (OU)**, simile a un filesystem di cartelle
- Centralizza la fatturazione (consolidated billing)
- Policy disponibili: **SCPs**, backup policies, tag policies
- Permette il consolidamento di Reserved Instances e Savings Plans

#### AWS Control Tower

Livello superiore rispetto a Organizations:
- Tutto ciò che offre Organizations, più:
  - **Account templates** — baseline automatica di un account dopo la creazione
  - **Guardrails** — prevenzione di azioni non consentite nell'organizzazione
  - Integrazione con **IAM Identity Center** (ex AWS SSO)
  - **Landing Zone** — insieme di account e OU pre-configurati con CloudTrail per l'organizzazione e capacità di audit di sicurezza

#### IAM Identity Center (ex AWS SSO)

Due istanze possibili:

| Tipo | Caratteristiche |
|------|-----------------|
| **Organization instance** | La versione completa. Gestisce accesso federato a più account. Richiede Organizations. Supporta solo SAML. |
| **Account instance** | Versione ridotta ("crippled"). Si abilita automaticamente quando si crea un IAM user con le impostazioni default nella console. Non funziona come la versione Organization. |

IAM Identity Center usa internamente IAM roles (gestiti automaticamente, non creati a mano).

---

### Federazione dell'accesso AWS

#### SAML 2.0 (per singoli account)

Provider supportati: Active Directory, Azure Entra ID, Google Workspace, Okta, ecc.
Meccanismo: traduzione dall'identità esterna a un IAM Role.

#### OIDC (OpenID Connect)

Caso d'uso principale: federare **GitHub Actions** con AWS, permettendo ai workflow CI/CD di eseguire task AWS con credenziali temporanee.

#### IAM Identity Center (per multi-account)

- Supporta solo SAML (non OIDC)
- Alternativa: directory interna (default di Control Tower)
- Gestisce i ruoli IAM in tutti gli account automaticamente

---

### S3 — Simple Storage Service

#### S3 Gateway Endpoint

- Permette l'accesso privato a S3 da una VPC senza passare per Internet
- **Non può essere usato transitivamente** — non funziona cross-region
- Da preferire sempre quando si ha traffico S3 interno a un'unica region

#### S3 Interface Endpoint (PrivateLink)

- Alternativa al Gateway Endpoint
- **Può essere usato transitivamente** — funziona per accesso cross-region
- Recente aggiunta (circa marzo 2025): ora supporta servizi cross-region direttamente
- Più costoso del Gateway Endpoint

🎯 **Differenza chiave per l'esame:** Gateway endpoint = non transitivo, Interface endpoint = transitivo.

#### Cifratura S3

| Metodo | Responsabilità chiave | Note |
|--------|-----------------------|------|
| SSE-S3 | AWS gestisce tutto | Nessuna visibilità sul processo di cifratura |
| SSE-KMS | KMS (AWS managed o customer managed) | Migliore controllo, auditing via KMS |
| SSE-C | Cliente fornisce la chiave al momento dell'upload | AWS cifra con la chiave fornita |
| Client-side encryption | Cliente cifra prima dell'upload | Massimo controllo, più complesso |

🎯 **Per l'esame:** All'Associate level, la risposta corretta è sempre la **soluzione managed AWS**, non la soluzione custom. Quindi se si deve scegliere tra SSE-KMS e client-side encryption, la risposta corretta per il contesto dell'esame è SSE-KMS.

**Nota su KMS vs CloudHSM:**
- **KMS** = completamente virtuale, non hardware-based
- **CloudHSM** = hardware-based (Hardware Security Module fisico)

#### S3 ACL (Access Control List)

- Formato XML, legacy, in deprecazione attiva
- Nessuna granularità (accesso o non accesso)
- Non implementa least privilege
- **Eliminare sempre nelle domande d'esame** a meno che non sia l'unica soluzione tecnicamente possibile

#### S3 Replication Time Control (RTC)

Checkbox aggiuntivo nella configurazione della replica S3:
- Garantisce che il 99% (o 99.9%) degli oggetti venga replicato entro **15 minuti**
- Utile per soddisfare RPO di 15 minuti in scenari di disaster recovery

---

### VPC — Virtual Private Cloud

#### Architettura di base

Componenti fondamentali:
- **Public Subnet** — per risorse con accesso internet (Load Balancer, NAT Gateway)
- **Private Subnet** — per compute (EC2, ECS), database, risorse interne
- **Internet Gateway** — connettività internet per subnet pubbliche
- **NAT Gateway** — permette alle risorse in private subnet di accedere a internet in uscita

#### Security Groups vs Network ACL

| Caratteristica | Security Groups | Network ACL |
|----------------|-----------------|-------------|
| Livello | Istanza (ENI) | Subnet |
| Stato | Stateful | Stateless |
| Regole multiple | Union delle regole (la più permissiva vince) | Ordine numerico, prima regola che fa match |
| Impatto | Solo l'istanza | Tutto il traffico della subnet |

🎯 **Per l'esame:** Se bisogna isolare un'istanza compromessa senza impattare altri workload della stessa subnet, usare **Security Group** (non Network ACL). Sostituire il security group con uno restrittivo — aggiungere un secondo security group vuoto NON funziona perché si prende l'unione delle regole.

Nota avanzata (non richiesta all'esame): sostituire un security group non taglia immediatamente le connessioni TCP già stabilite.

#### VPC Endpoints

**Gateway Endpoint:**
- Per S3 e DynamoDB
- Gratuito
- Non transitivo (non usabile cross-region)
- Si aggiunge una route alla route table della subnet

**Interface Endpoint (PrivateLink):**
- Per la maggior parte degli altri servizi AWS
- Costo aggiuntivo
- Transitivo — usabile cross-region
- Recente: ora supporta S3 cross-region direttamente

#### VPC Block Public Access

Funzionalità relativamente nuova: guardrail a livello di VPC che impedisce qualsiasi configurazione troppo permissiva, override di qualsiasi regola che consentirebbe accesso pubblico indesiderato.

#### Connettività tra VPC

| Soluzione | Quando usarla |
|-----------|---------------|
| **VPC Peering** | Piccolo numero di VPC, connettività punto-a-punto |
| **Transit Gateway** | Mesh di connettività tra molte VPC, anche cross-region (richiede TGW peering tra region) |
| **VPC Lattice** | Espone singole interfacce/listener a molti consumer — più granulare del TGW, audit centralizzato |
| **PrivateLink + Interface Endpoint** | Esporre un servizio specifico a consumer in VPC diverse |

**VPC Peering:** Non transitivo — se A-B e B-C sono in peering, A non può raggiungere C attraverso B.

**Transit Gateway cross-region:** Richiede Transit Gateway in ogni region + TGW Peering connections tra di essi.

**VPC Lattice:** Come PrivateLink ma in modalità mesh (molti servizi esposti, molti consumer). Ideale quando il numero di connessioni diventa difficile da tracciare con i singoli Interface Endpoint.

#### SSM Session Manager vs SSH

Anti-pattern: accesso SSH diretto o tramite bastion host.

Alternativa raccomandata: **AWS Systems Manager Session Manager**
- Il SSM Agent sull'istanza crea una connessione WebSocket verso il servizio SSM
- Accesso interattivo al terminale senza aprire porte inbound nel Security Group
- Richiede: SSM Agent installato + Instance Profile con permessi SSM + (idealmente) Interface Endpoint per SSM per traffico completamente privato

---

### GuardDuty

**Definizione:** Servizio di threat detection basato su ML.

**Data source di default (ingeste automaticamente, anche se non abilitate):**
1. CloudTrail logs
2. VPC Flow Logs
3. VPC DNS logs

**Funzionamento:** Costruisce un modello ML del comportamento normale dell'account. Comportamenti anomali generano **findings**.

**Integrazioni:**
- Security Hub (centralizzazione dei finding)
- EventBridge (per remediation automatica)
- Amazon Detective (analisi forense)

**Delegated Administrator:** Con Organizations, un account può essere designato come delegated administrator per GuardDuty — tutti i finding convergono in un unico posto.

**Costo:** Amazon Detective (analisi forense) può essere costoso in ambienti con alta rotazione di risorse.

---

### Security Hub

**Hub centrale** per tutti i finding di sicurezza.

Integrazioni principali:
- GuardDuty
- AWS Config
- Amazon Inspector
- Macie
- Firewall Manager

Output di Security Hub:
- Audit Manager (report di audit)
- EventBridge (alerting e remediation)
- Amazon Detective (analisi forense)

---

### AWS Config

**Definizione:** Servizio di compliance e configuration management — NON un servizio di sicurezza primario.

**Cosa fa:**
- Registra le configurazioni delle risorse nel tempo (recording stream)
- Valuta la conformità rispetto a regole definite (Config Rules)
- Segnala risorse non conformi
- Supporta remediation automatica via SSM Automation documents

**Alerting:** Config non ha notifiche native robuste — necessita di EventBridge per creare allarmi su eventi di conformità.

**Scalabilità:**
- Config Rule normale: deve essere deploiata in ogni region di ogni account (problema di scala)
- **Conformance Pack**: bundle di regole deploiabili insieme tramite CLI con JSON input (riduce il numero di operazioni)
- **Delegated Administrator + Organizations**: unico punto di raccolta per tutti gli account, auto-onboarding dei nuovi account — soluzione scalabile

**Config vs Inspector vs CloudWatch vs Trusted Advisor:**
- **Config** = valutazione della configurazione delle risorse, compliance
- **Inspector** = vulnerabilità software e OS (non configurazione)
- **CloudWatch** = metriche e log (non configurazione)
- **Trusted Advisor** = check limitati, nessuna notifica, nessuna flessibilità

---

### Amazon Inspector

**Scopo:** Scansione di vulnerabilità software e OS su EC2, container, Lambda.

Non valuta la configurazione delle risorse (quello è Config). Utile in architetture Defense in Depth per monitorare le istanze EC2 in esecuzione.

---

### CloudFormation

**Caso d'uso chiave per l'esame (sicurezza):**

Se la domanda chiede "come garantire che tutti i controlli di sicurezza vengano applicati consistentemente sin dal deployment, minimizzando l'errore umano" — la risposta è **CloudFormation** (o CDK).

CloudFormation bake-in tutti i requisiti di sicurezza negli stack, garantendo consistency e riducendo human error. CloudFormation Stack Sets permettono di gestire deployment multi-region e multi-account da un unico posto.

---

### AWS CloudTrail

- Registra tutte le API call verso i servizi AWS
- Ogni richiesta viene valutata dal permission evaluation engine e il risultato diventa un log CloudTrail
- Si può creare un **Trail per tutta l'organizzazione** (soluzione scalabile)
- I log possono essere inviati a CloudWatch Logs, S3, o altri target

---

### RDS e Aurora

#### RDS

**Operational overhead eliminato rispetto a DB su EC2:**
- Backup automatici
- Replica
- Software updates
- Failover automatico
- Restore
- Scaling

RDS è più costoso di EC2 per la stessa CPU/RAM, ma il costo del personale per gestire un DB su EC2 rende RDS più conveniente nel totale.

#### Aurora

- Compatibile con MySQL e PostgreSQL
- Single Writer + multiple Readers con failover automatico
- **Aurora Serverless**: elimina writer/reader fissi, scala automaticamente usando ACU (Aurora Compute Units). Non adatto a query molto grandi.
- **Aurora Global Database**: primary in una region, read replica in un'altra. Permette write dalla region secondaria (proxied alla primary). Sub-secondo latency di replica.
- **Aurora DSQL**: distributed database active-active multi-region (molto nuovo, Chad non l'ha sperimentato)

**Cross-region Read Replica RDS:** Latency di replica variabile in base a sizing e carico — monitorare il replica lag per garantire RPO.

🎯 **Per l'esame:** Aurora è quasi sempre la risposta "migliore" rispetto a RDS standard per scenari che richiedono alta disponibilità o scalabilità.

---

### DynamoDB

- Database NoSQL fully managed
- **DynamoDB Streams:** stream di eventi per ogni modifica alla tabella
- **Global Tables:** replica multi-region bi-direzionale — propagano anche le DELETE
- Replica tra region in Global Tables tipicamente entro **1 secondo**

**Caso d'uso Disaster Recovery:**
Se il requisito è "non propagare i delete al region secondario" — Global Tables non vanno bene (propagano i delete). La soluzione corretta è:
1. DynamoDB Streams abilitato sulla tabella primaria
2. Lambda function nella region secondaria che consuma lo stream e applica solo CREATE e UPDATE (non DELETE)

**DynamoDB per session state:**
In un'applicazione EC2 con Auto Scaling, lo stato della sessione (es. carrello acquisti) non deve essere tenuto in memoria locale sull'istanza — va esternalizzato in DynamoDB o ElastiCache. Questo rende l'applicazione stateless e compatibile con lo scaling.

---

### ECS ed EKS

**ECS su Fargate:** elimina la gestione dell'infrastruttura EC2 sottostante. Auto-scaling disponibile.

**EKS su Fargate:** Kubernetes managed senza EC2.

Alternativa peggiore: Docker su EC2 self-managed — richiede la stessa gestione operativa di un DB su EC2.

---

### EFS — Elastic File System

- Sistema di file NFS managed e **truly elastic**: non si provisiona la dimensione, si espande/contrae automaticamente
- Alternativa a NFS self-managed su EC2 (che richiede replica, gestione dello spazio disco, ecc.)

---

### Route 53

#### Health Checks

Tipi di sorgenti supportate per i Route 53 health check:
- Endpoint raggiungibili pubblicamente (public IP o DNS)
- CloudWatch Alarms (singoli o aggregati)
- **NON supporta CloudWatch Metrics direttamente** (solo Alarms)

#### Route 53 Application Recovery Controller (ARC)

Servizio per il failover automatico tra regioni:
- Crea **Readiness Checks** (essenzialmente Route 53 health check sull'ALB) per ogni region
- Quando la primary region fallisce il readiness check, ARC migra il traffico alla region secondaria **in pochi minuti**

#### Routing Policies utili

- **Latency-based routing con health check:** invia il traffico al load balancer con minor latenza dal client — utile per deployment multi-region attivo-attivo

---

### CloudWatch

#### Quattro fasi del monitoring

1. **Generate:** dove si producono le metriche
   - Metriche intrinseche di EC2, ECS, ELB, RDS
   - Custom metrics (push manuale o tramite CloudWatch Agent)
   - Personal Health Dashboard + EventBridge per eventi a livello di infrastruttura AWS

2. **Aggregate:** dove si raccolgono i log
   - **CloudWatch Logs** è il principale aggregatore
   - Fonti: CloudTrail, API Gateway, ECS/EKS container logs, Lambda execution logs, RDS database logs, SSM Run Command output, EC2 OS/application logs via CloudWatch Agent

3. **Process:** come si reagisce
   - **CloudWatch Alarms** per metriche numeriche — possono triggherare Auto Scaling o Lambda
   - **EventBridge** per log ed eventi (event-driven)
   - SNS per notifiche (email, SMS, webhook Jira)
   - AWS Chatbot per Slack/Teams (supporta anche risposte interattive)

4. **Store and Analyze:**
   - CloudWatch Logs può avere TTL configurabile
   - Costoso per storage a lungo termine — meglio esportare in S3 via Data Firehose o OpenSearch
   - CloudWatch Log Insights per query sui log
   - CloudWatch Log Insights query convertibili in Alarms

**KPI vs metriche AWS:** Le metriche AWS intrinseche (CPU, rete) non misurano il valore di business. I KPI veri spesso richiedono custom metrics.

---

### CloudWatch Synthetics Canaries

- Lambda functions che girano all'interno della VPC
- Eseguono health check sofisticati: possono colpire multiple URL, seguire workflow, fare screenshot, raccogliere telemetria
- Utili per applicazioni in **private subnet** (non raggiungibili da Internet)
- Permettono di calcolare l'availability dell'applicazione e di integrare con Route 53 health check tramite CloudWatch Alarm

---

### AWS Backup

- Servizio centralizzato per gestire backup di risorse AWS
- Supporta copia verso regioni remote (cross-region backup)
- Alternativa a soluzioni custom per la replica di AMI e snapshot

---

### EC2 Image Builder e Data Lifecycle Manager

Strumenti alternativi per automatizzare la creazione e copia di AMI verso regioni di failover.

---

### AWS Chatbot

- Integrazione con Slack e Microsoft Teams
- **Supporta comunicazione bidirezionale:** oltre a ricevere notifiche, permette di rispondere nel canale Slack per invocare comandi AWS CLI (operazioni read-only)
- Chad lo preferisce all'email per le notifiche (più efficace)

---

### Amazon Detective

- Analisi forense di incident di sicurezza
- Genera heat map geografiche degli eventi anomali
- Potenzialmente costoso in ambienti con alta rotazione di risorse — monitorare

---

### AWS Macie

- Identificazione e protezione di dati sensibili in S3 (PII, dati finanziari, ecc.)
- Si integra con Security Hub

---

### Lambda

**Caratteristiche rilevanti:**
- Può essere triggherata da DynamoDB Streams, EventBridge, S3 events, CloudWatch Alarms
- **Cold start:** problema reale ma mitigabile con:
  - Provisioned Concurrency (warm pools)
  - Traffico costante (mantiene la funzione warm)
  - Uso di Docker container (riduce il cold start in certi casi — contro-intuitivo ma verificato da Chad)
- Può eseguire all'interno di una VPC (per accedere a risorse private)
- **CloudWatch Synthetics Canaries** sono Lambda functions specializzate

---

## Architettura e Design — Pattern Discussi

### Defense in Depth (Difesa in Profondità)

Applicare security a tutti i livelli appropriati. Esempio con applicazione EC2 + DynamoDB:

1. **Rete:** VPC con public/private subnet, NAT Gateway, ALB nella public subnet
2. **Compute:** EC2 in private subnet, Inspector abilitato, security group con least privilege ingress (solo dall'ALB sulla porta applicativa)
3. **Accesso al sistema:** SSM Session Manager invece di SSH (no porte inbound)
4. **Routing:** VPC Gateway Endpoint per DynamoDB (traffico privato)
5. **IAM:** Instance Profile con IAM Role per DynamoDB, condizione che limita le richieste al VPC ID specifico
6. **Permission Boundary:** sulla policy per prevenire espansione non intenzionale dei permessi
7. **Endpoint Policy:** sul Gateway Endpoint, limita il traffico alla subnet privata
8. **SCP Organizations:** nega accesso alla tabella DynamoDB eccetto tramite il Gateway Endpoint
9. **Cifratura:** KMS per cifrare la tabella, con key policy che ammette solo questo IAM Role
10. **DynamoDB Resource Policy:** rinforza i permessi

Nota importante: non è necessario implementare tutti i livelli — scegliere quelli easy to implement, audit e monitor. Troppi livelli rendono il troubleshooting un incubo.

---

### Architettura Resiliente — Decoupling

**Pattern: da monolitico a decoupled**

Prima (tightly coupled):
```
EC2 instance monolitica (tutti i tier) → Public DNS
```

Dopo (decoupled):
```
ALB (multi-AZ) → Auto Scaling Group (EC2) → Aurora Serverless
```

Vantaggi:
- Ogni tier scala indipendentemente
- Failover automatico anche con fallimento di 2 AZ nella stessa region
- ALB richiede **almeno 2 AZ** (non funziona con single-AZ)

---

### Sizing delle istanze nell'Auto Scaling Group

Principio spesso trascurato: **prefer many small instances over few large instances**.

Esempio numerico:
- 3 istanze 8xlarge → se una cade, impatto su 33% delle richieste
- 27 istanze large (stesso costo mensile) → se una cade, impatto su 4% delle richieste

Benefici aggiuntivi del maggior numero di istanze:
- Ogni scale-out aumenta meno il costo
- Granularità migliore nel scaling

---

### Disaster Recovery — Scenario Completo

**Requisiti:** RTO 1 ora, RPO 15 minuti. Primary: us-east-1, Recovery: us-west-2.

**Infrastruttura primaria:**
- ALB + Auto Scaling Group (EC2) per web/app tier
- RDS (o Aurora) per database relazionale
- DynamoDB per database NoSQL
- S3 per storage documenti

**Replica verso us-west-2:**

| Risorsa | Meccanismo di replica |
|---------|----------------------|
| S3 | Bucket Replication + S3 Replication Time Control (99% entro 15 min) |
| DynamoDB | Global Tables (replica entro 1 secondo) |
| RDS | Cross-region Read Replica (< 1 secondo, ma monitorare il lag) |
| Aurora | Aurora Global Database (alternativa migliore a RDS) |
| EC2 AMI | AWS Backup, EC2 Image Builder, Data Lifecycle Manager, o EventBridge+Lambda |
| ALB + ASG + EC2 | CloudFormation/CDK Stack (o Stack Sets per multi-region) |

**DNS Failover:** Route 53 Application Recovery Controller (ARC) con Readiness Checks.

**Verifica RTO/RPO:**
- RTO 1 ora: ARC migra il traffico in pochi minuti → ampiamente rispettato
- RPO 15 minuti: S3 RTC garantisce 15 min, DynamoDB < 1 secondo, RDS dipende dal lag

---

### Architettura di Monitoring

**Fase Generate → Aggregate → Process → Store/Analyze**

Flusso tipico:
```
EC2/ECS/RDS/Lambda metrics → CloudWatch Metrics/Logs
CloudTrail → CloudWatch Logs / S3
VPC Flow Logs → CloudWatch Logs / S3
App logs (CloudWatch Agent) → CloudWatch Logs
                    ↓
CloudWatch Log Insights queries
CloudWatch Alarms (su metriche numeriche)
EventBridge Rules (su log/eventi)
                    ↓
SNS → email/SMS/webhook
Chatbot → Slack/Teams
Lambda → remediation automatica
Auto Scaling → scale out/in
                    ↓
Data Firehose → S3 (storage economico a lungo termine)
              → OpenSearch (analisi)
```

---

## Demo e Esempi Pratici

### Esempio 1: Accesso privato cross-region a S3

**Scenario:** EC2 in private subnet us-west-2, accesso a S3 in ap-southeast-1, traffico completamente privato.

**Soluzione corretta (al momento dell'esame):**
- B: VPC in ap-southeast-1 con peering dalla VPC principale
- D: Interface Endpoint per S3 nella VPC ap-southeast-1
- F: Configurare l'applicazione per usare l'endpoint regionale ap-southeast-1

**Perché si eliminano A e C (Gateway Endpoint):** I VPC Gateway Endpoint **non sono transitivi** — non funzionano cross-region.

**Update realworld (post-esame):** Ora le Interface Endpoint supportano servizi cross-region direttamente — basterebbe un Interface Endpoint per S3 puntato alla region ap-southeast-1.

---

### Esempio 2: Networking a livello globale

**Scenario iniziale (anti-pattern):** VPC in 3 region, alcune connessioni in peering, altre via public IP/Internet Gateway.

**Miglioramenti raccomandati:**
1. Sostituire tutte le connessioni public IP con VPC Peering
2. Per mesh completa tra molte VPC: Transit Gateway in ogni region + TGW Peering tra region
3. Per esporre servizi specifici: VPC Lattice (audit centralizzato, granularità a livello di interfaccia/listener)

---

### Esempio 3: Tokenizzazione di dati PII da EFS su VPN

**Scenario:** Applicazione on-prem, dati PII su EFS raggiungibile via site-to-site VPN. Ogni file deve avere certi campi tokenizzati il più rapidamente possibile. La tokenizzazione deve avvenire interamente in AWS.

**Chiave per eliminare le risposte:**
- Eliminare le risposte con "batch" o "EMR" — non soddisfano "as quickly as possible"
- Rimanere con le risposte basate su Lambda (event-driven, bassa latency)

**Definizione di tokenizzazione:** Non è cifratura inline. È la **sostituzione** di un valore sensibile con un token casuale, con il valore originale archiviato separatamente indicizzato dal token.

**Risposta corretta:** Lambda triggerata da S3 event, che legge il file, genera token per i campi sensibili, archivia i valori originali in DynamoDB con il token come chiave, riscrive il file con i token al posto dei valori sensibili.

---

### Esempio 4: Stato della sessione in Auto Scaling

**Scenario:** Applicazione eCommerce stateful su EC2 con Auto Scaling. Durante gli eventi di scaling, il carrello acquisti si svuota.

**Causa:** Lo stato del carrello è memorizzato in memoria locale sull'istanza EC2. Quando l'ALB reindiriz il traffico a un'altra istanza (default: least outstanding requests), lo stato è perso.

**Soluzioni testate:**
- Steady state group (no scaling): non risolve (hardware failure, deploy di aggiornamenti causano comunque replacement)
- ALB Sticky Sessions: parziale, non risolve il problema di scaling
- **Soluzione corretta:** Esternalizzare lo stato del carrello in DynamoDB e referenziare la tabella invece della cache locale

---

### Esempio 5: Monitoraggio applicazione in private subnet

**Scenario:** Applicazione EC2 su porta 8080 in private subnet. Monitorare per calcolare availability e supportare failover automatico.

**Analisi delle opzioni:**
- Cron job con script: tecnicamente funziona ma il cron è intrinsecamente inaffidabile
- CloudWatch Agent: non supporta custom metrics direttamente (solo metriche di sistema)
- **CloudWatch Synthetics Canary** (risposta C): Lambda nella VPC, può colpire l'app su porta 8080, sofisticato, calcola availability → genera CloudWatch Alarm → Route 53 Health Check può usare il Alarm
- Route 53 Health Check su CloudWatch Alarm (risposta D): Route 53 supporta Alarms come sorgente per health check, supporta failover automatico

**Risposte B ed E eliminate:** Route 53 health check non supporta CloudWatch Metrics direttamente — solo Alarms.

---

## Trade-off e Limitazioni

### RDS vs Aurora

| Aspetto | RDS | Aurora |
|---------|-----|--------|
| Motori supportati | MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, Db2 | MySQL, PostgreSQL |
| Costo | Inferiore | Superiore |
| Failover | Automatico (Multi-AZ) | Automatico, più veloce |
| Read Replica scalabilità | Limitata | Fino a 15 read replica con auto-scaling |
| Cross-region | Cross-region Read Replica (unidirezionale) | Aurora Global Database (con write forwarding) |
| Serverless | Solo RDS su Graviton | Aurora Serverless v2 |
| Per l'esame | Menzionato | Quasi sempre la risposta migliore per HA |

### DynamoDB Global Tables vs Lambda+Streams per DR

| Aspetto | Global Tables | Lambda+Streams |
|---------|--------------|----------------|
| Direzione replica | Bidirezionale | Configurabile (unidirezionale) |
| Propagazione DELETE | Si | Configurabile (può escludere) |
| Latenza replica | < 1 secondo | Secondi (dipende dalla Lambda) |
| Complessità | Bassa | Alta |
| Uso corretto | DR generale | Quando non si vogliono propagare i delete |

### Security Groups vs Network ACL

| Aspetto | Security Group | Network ACL |
|---------|----------------|-------------|
| Granularità | Istanza | Subnet intera |
| Regole multiple | Union (più permissivo vince) | Prima regola che fa match |
| Stato | Stateful | Stateless |
| Impatto su altri workload | No | Si (tutta la subnet) |
| Per isolare istanza compromessa | **Preferito** | Sconsigliato |

### VPC Gateway Endpoint vs Interface Endpoint

| Aspetto | Gateway Endpoint | Interface Endpoint |
|---------|-----------------|-------------------|
| Servizi | S3 e DynamoDB | Quasi tutti i servizi AWS |
| Costo | Gratuito | A pagamento |
| Cross-region | No | Si (recente) |
| Transitività | No | Si |
| Implementazione | Route nella route table | ENI nella subnet |

### SSE-S3 vs SSE-KMS vs SSE-C vs Client-side

Già coperto nella sezione S3. In breve:
- Massimo controllo del cliente = Client-side encryption
- Ma per l'esame: sempre preferire la soluzione managed AWS (SSE-KMS)

### RBAC vs ABAC

Già coperto nella sezione IAM.

---

## Sicurezza e Considerazioni Operative

### Principio: Implement a Strong Identity Foundation

**Elementi chiave:**
1. Eliminare long-term static credentials (access key di IAM user) dove possibile
2. Usare IAM Roles e credenziali temporanee
3. Preferire ABAC a RBAC per least privilege scalabile
4. MFA obbligatorio sull'account root e sugli IAM user
5. Federation con identity provider esterni

### Principio: Enable Traceability

**Stack di monitoring per security:**
GuardDuty → Security Hub → Audit Manager / EventBridge / Amazon Detective

### Principio: Apply Security at All Layers (Defense in Depth)

Dettagliato nella sezione Architettura e Design.

### Principio: Automate Security Best Practices

**Config per la governance automatizzata:**
- Config Rules per valutare compliance delle risorse
- SSM Automation documents per remediation automatica
- Conformance Pack per bundle di regole
- Config con Delegated Administrator per governance multi-account

**Scalabilità:**
- CloudTrail trail di organizzazione — scalabile
- GuardDuty con delegated administrator — scalabile
- Config con delegated administrator — scalabile (per region, non per account singoli)

### Principio: Protect Data in Transit and at Rest

**Data classification prima di tutto:** capire quale dato è pubblico, privato, confidenziale, prima di implementare qualsiasi protezione.

**Esempio ABAC per la protezione dei dati in S3:**
1. Taggare ogni oggetto S3 con il livello di sensitivity
2. S3 bucket policy che impone il tagging durante l'upload
3. KMS key policy che permette la cifratura solo di oggetti con i tag corretti
4. IAM User/Group taggati con il loro livello di accesso autorizzato
5. S3 bucket policy che verifica sia il tag dell'oggetto che il tag del richiedente
6. Organizations tagging policy che impone la presenza dei tag su tutte le risorse

### Principio: Keep People Away from Data

**Anti-pattern:**
- SSH pubblico diretto a EC2
- Bastion host
- Accesso diretto al database di produzione dal laptop dello sviluppatore

**Alternative:**
- SSM Session Manager (no SSH, no porte inbound)
- QuickSight per query predefinite su dati
- API Gateway + ORM come layer di accesso al database (non SQL diretto)

**Tecniche di obfuscazione:**
- Anonimizzazione tramite hashing
- Pseudonimizzazione (splitting di dati sensibili in storage diverso, token come riferimento)

**Nota su algoritmi:** Usare algoritmi state-of-the-art per hashing e cifratura, evitare algoritmi noti come insicuri (ancora molto diffusi).

### Principio: Prepare for Security Events

**Ciclo di incident response (tre fasi):**

**PREVENT (verde):**
- Stabilire policy e guardrail
- Progettare l'architettura con prevenzione in mente
- Limitare il blast radius (segregazione in VPC e account separati)
- Infrastructure as Code per self-documenting infrastructure
- Network Access Analyzer, IAM Access Analyzer, Security Hub, Config

**RECOGNIZE (giallo):**
- Stabilire un baseline di comportamento normale
- Configurare alert e monitoring: CloudWatch, GuardDuty, WAF, VPC Flow Logs
- Usare AWS Chatbot per notifiche interattive in Slack/Teams

**MITIGATE (rosso):**
- Isolation, scaling, delete di risorse
- Playbook e runbook predefiniti (es. sostituire security group, ruotare chiavi)
- Backup/snapshot di risorse compromesse (evidence retention + replay per test)
- Il ciclo non ha fine: tutto ciò che si impara nella fase Mitigate torna nella fase Prevent

---

## Q&A Notevoli

**D: Si dovrebbe fare il Solutions Architect Pro prima o dopo l'Associate?**
R: Prima fare Solutions Architect Associate. Poi, se si vuole il Pro, procedere. Ma per tutti, indipendentemente dalla disciplina, l'Associate viene prima.

**D: Differenza tra Reliability e Resiliency?**
R: Semanticamente interscambiabili per Chad, anche se AWS non è sempre consistente nell'uso. Possible distinzione: reliability = connessa a availability e scalabilità; resilience = capacità sotto condizioni avverse (failure, ecc.).

**D: Dopo questo esame: Security Specialty o SysOps?**
R: Dipende dall'obiettivo. Security Specialty è più un resume builder universale. SysOps testa l'esperienza hands-on operativa in modo che gli altri esami non fanno.

**D: Differenza tra Monitoring e Observability?**
R di Chad: Observability = avere implementato molti metriche e dashboard disponibili. Monitoring = identificare le metriche critiche, implementare soglie e pipeline per valutarle e agire (alert o remediation).

**D: Vale la pena fare la repatriation on-prem per ridurre i costi?**
R: La repatriation funziona solo a scala molto grande (Netflix, Meta). Per aziende più piccole, l'analisi di TCO tipicamente trascura il costo del personale, la perdita di agilità e il costo dell'innovazione non fatta.

**D: Reasonable balance tra security e effort?**
R: Dipende dai requisiti. La security è uno spectrum: più security = meno usabilità e più overhead operativo. Implementare solo i controlli richiesti. La documentazione della decisione è fondamentale (se la scelta è accettare un certo rischio per maggiore agilità, documentarlo esplicitamente). Si può migliorare iterativamente.

**D: Lambda cold start è un problema per la tokenizzazione "as quickly as possible"?**
R: No, perché ci sono mitigazioni: Provisioned Concurrency, funzioni già warm per traffico costante, container Docker (riduce il cold start). Qualche secondo è accettabile per la tokenizzazione.

---

## Risorse e Riferimenti

- **AWS Glossary** — dizionario di termini AWS (link condiviso in chat)
- **AWS Well-Architected Framework** — white paper ufficiale per ogni pillar (link: aws.amazon.com/architecture/well-architected)
- **AWS Decision Guides** — guide per scegliere tra servizi dello stesso dominio (database, storage, networking, ecc.)
- **Chad Smith su O'Reilly** — pagina di ricerca con tutti i suoi corsi
- **Security Specialty Crash Course** (Chad Smith su O'Reilly) — approfondimento sicurezza AWS
- **Practice Tests SAA-C03** (Chad Smith su Udemy) — due test: uno facile (fondamentali), uno difficile (bordering on Pro level)
- **Chad Smith su LinkedIn** — per domande post-corso

---

## Takeaway Applicabili

1. **Leggere i white paper del Well-Architected Framework**, specialmente Cost Optimization e Security, prima dell'esame.

2. **Non fare mai l'errore degli ACL:** nelle domande d'esame, eliminare sempre le risposte con ACL S3, a meno che non siano l'unica soluzione tecnica possibile.

3. **Managed service > unmanaged sempre** (al livello Associate): se le scelte sono tra soluzione managed AWS e soluzione custom, scegliere sempre la managed.

4. **Instance Profile != IAM Role:** con Terraform, bisogna creare esplicitamente l'Instance Profile come risorsa separata.

5. **Gateway Endpoint non è transitivo:** non può servire S3 cross-region. Usare Interface Endpoint (o VPC secondaria nella region di destinazione).

6. **Security Group e unione delle regole:** aggiungere un secondo security group vuoto non "isola" un'istanza — si prende l'unione di tutte le regole. Bisogna sostituire il security group.

7. **Network ACL impatta tutta la subnet:** non usare NACLs per isolare una singola istanza compromessa (impatto su altri workload).

8. **DynamoDB Global Tables propagano le DELETE:** in scenari DR dove le delete non devono propagarsi, usare DynamoDB Streams + Lambda unidirezionale.

9. **Route 53 health check supporta CloudWatch Alarms ma NON CloudWatch Metrics direttamente.**

10. **SSM Session Manager è sempre preferibile a SSH** per l'accesso alle istanze EC2 — elimina l'esposizione di porte inbound.

11. **La cost optimization è una strategia aziendale**, non una caccia alle risorse idle — senza visibilità e attribuzione dei costi, si è sempre in modalità reattiva.

12. **RTO 1h / RPO 15 min:** configurazione DR standard nell'esame — Aurora Global Database + S3 RTC + DynamoDB Global Tables + Route 53 ARC.

13. **Prefer many small instances over few large instances** in Auto Scaling Group per ridurre l'impatto del fallimento di una singola istanza (33% → 4% delle richieste).

14. **ABAC scala meglio di RBAC** per least privilege, ma è più complesso da progettare.

15. **CloudFormation/CDK = consistency e riduzione human error** — risposta giusta quando la domanda chiede come garantire l'applicazione costante dei controlli di sicurezza sin dal deployment.

---

## Note sulla Trascrizione

### Copertura
- **Copertura temporale:** 100% del corso (~3.9h su ~3.9h trascritti)
- **Copertura tematica:** completa per la sessione del Giorno 1
- Questa è sessione 1 di 2 — il corso completo è 8h. Il Giorno 2 coprirà Reliability (completamento), Performance Efficiency e Cost Optimization

### Qualità della trascrizione
- La trascrizione è generata automaticamente (probabile Whisper o simile)
- Alcuni errori di trascrizione notevoli:
  - "Aurora server list" = Aurora Serverless
  - "AOB" = ALB (Application Load Balancer)
  - "ECUs" = ACUs (Aurora Compute Units)
  - "JIG" = potrebbe essere "CDK" o altro tool IaC
  - "Consistence Manager" = Systems Manager
  - "varsity events" / "varsity stream" = "filter events" / "filter stream" (probabile)
- Presenza di ripetizioni e falsi inizi tipici del parlato spontaneo

### Sezioni mancanti
- Aggiornamenti al giorno 2 (Performance Efficiency, Cost Optimization completo)
- Alcune parti delle slide non sono descritte verbalmente (erano animate e non funzionanti)
- I link condivisi in chat non sono presenti nella trascrizione (solo menzioni verbali)
- Il testo delle domande d'esame è parzialmente leggibile ma senza le singole opzioni di risposta (solo la lettera menzionata verbalmente)

### Affidabilità
Alta per i concetti tecnici e i pattern architetturali. Bassa per i dettagli numerici specifici (es. dimensioni esatte delle policy inline, percentuali precise di S3 RTC). Verificare sempre con la documentazione ufficiale AWS per i valori precisi.

---

---

# GIORNO 2 — Contenuto aggiuntivo (Iterazione 2)

**Data sessione:** successiva al Day 1  
**Copertura:** 13.434 secondi (~3h 44m), 100%  
**Facilitatore:** Andrew Eric (O'Reilly Media), Istruttore: Chad Smith

---

## Reliability Pillar — Principi Completati (Day 2)

### Principio: Stop Guessing Capacity

La capacità in AWS scala automaticamente per molti servizi chiave:

| Servizio | Come scala |
|---|---|
| **S3** | Performance scala con la domanda; hard limits sono molto distanti per workload normali |
| **Lambda** | Concorrenza configurabile; scala on-demand |
| **CloudFront** | Cache scala con la distribuzione geografica del pubblico |
| **AWS Auto Scaling** | Gestisce EC2, ECS, Aurora Read Replicas, DynamoDB |

**Strategie per stop guessing capacity:**
1. **Automation**: usare servizi con auto-scaling nativo
2. **Obtain resources upon impairment detection**: Route 53 health checks, ELB health checks, CloudWatch alarms per scatenare scaling/failover
3. **Obtain resources upon demand detection**: AWS Auto Scaling (proactive + reactive) per EC2, ECS, Aurora Replicas, DynamoDB
4. **Load testing**: usare CloudFormation per deployare ambienti temporanei di test; alcuni team eseguono load test direttamente in produzione (richiede tagging nei log per separare il traffico reale dal test)

**CloudFront Edge Network (dettaglio aggiornato):**

| Tipo | Numero | Descrizione |
|---|---|---|
| CloudFront Points of Presence | 700+ | Data center AWS con hardware fisico e cache |
| CloudFront Embedded PoP | 900+ | Rack/cage embedded in data center ISP (più vicini all'utente finale) |
| Regional Edge Caches | 13 | Embedded in reti regionali — livello intermedio tra PoP e origin |

🎯 **Insight**: CloudFront supera come performance CDN specializzati come Cloudflare, grazie a questa infrastruttura capillare.

---

### Principio: Manage Change Through Automation

**Chiave**: non solo implementare i cambiamenti via automazione, ma anche gestire i cambiamenti all'automazione stessa come se fossero cambiamenti all'infrastruttura (tracking, review).

**Esempio 1 — Event-Driven con EventBridge + Lambda:**
```
AWS Health Service (hardware degradation event)
    → Personal Health Dashboard (visuale)
    → EventBridge (event stream)
        → Lambda (stop/start EC2 instance)
        — oppure —
        → Lambda (AWS SDK: scale-in via Auto Scaling Group, istanza sostituita da una nuova)
```

**Esempio 2 — Config + SSM Remediation:**
```
Requisito aziendale: RDS multi-AZ obbligatorio
    → AWS Config rule monitora compliance
    → Istanza non-compliant (multi-AZ=false)
    → Config remediation → SSM Automation Document
        → cambia multi-AZ da false a true
```

**Limitazione Config remediation:** funziona solo per cambiamenti booleani (true/false). Non supporta input testuali per la remediation.

---

### Deployment Types e Immutable Infrastructure

**Quattro tipi di deployment:**

| Tipo | Descrizione | Immutabile |
|---|---|---|
| **In-place** | Aggiornamento diretto sulle istanze esistenti | ❌ |
| **Rolling** | In-place ma fasato (es. 25% alla volta) | ❌ |
| **Blue-Green** | Due infrastrutture parallele, migrazione graduale del traffico | ✅ |
| **Canary** | Pre-deployment su piccola percentuale di risorse per validare | ✅ (parziale) |

**Immutable Infrastructure** = una volta deployata un'istanza, non viene mai modificata inline. L'unico modo di fare cambiamenti è sostituire l'intera istanza con una nuova che contiene l'ultima versione.

**Vantaggi dell'immutabilità:**
- ✅ **Security**: nessuna deriva dalla baseline sicura; aggiornamenti di sicurezza mancati non sono un problema
- ✅ **Simplified deployments**: nessuna complessità da gestire su deployment falliti (disk space esaurito, network transient, package non scaricabile, ecc.)
- ✅ **Scalability**: rollout e rollback più semplici
- ❌ **Performance** — **NON è un beneficio**: alcuni runtime si basano su cache (es. JVM, Memcached come sidecar) che diventano più performanti più a lungo l'istanza è in esecuzione. L'immutabilità può *ridurre* la performance in questi casi.

**Blue-Green Deployment — implementazioni possibili:**
- A livello **DNS** (Route 53 weighted routing, 100/0 → 50/50 → 0/100): si basa sui TTL DNS, potenzialmente lento
- A livello **Load Balancer** (cambio weights nel listener): più veloce del DNS
- A livello **Target Group** (due ASG in parallelo sul medesimo LB)
- A livello **Auto Scaling Group** (due Launch Template nel medesimo ASG)
- Applicabile anche a **RDS** (AWS ha feature native per blue-green RDS)

🎯 **Per l'esame**: Blue-Green è "favorito" da AWS. Separazione di competenze: il SA sa che è appropriato; il DevOps decide a quale livello implementarlo.

🎯 **Blue-Green e app stateful**: con LB stickiness, un blue-green a livello DNS può creare esperienza utente negativa. Meglio usare app stateless per blue-green.

---

## Performance Efficiency Pillar — Domain 3 (Completo)

### Principio: Democratize Advanced Technologies

**Significato pratico:** AWS diventa subject matter expert per tecnologie complesse (ML, NoSQL specializzati, streaming, IoT, ecc.) e le offre come managed service. Il team operativo del cliente non deve imparare 25 aspetti diversi di un tool — solo 2-3.

**Esempio concreto — TensorFlow su Docker:**

| Approccio EC2 | Approccio ECS Fargate |
|---|---|
| Launch EC2 + bootstrapping | Configura task definition |
| Installa Docker e dipendenze | Carica immagine su ECR |
| Deploya container | Configura servizio ECS |
| Configura monitoring/logging | Configura auto scaling |
| Monitora performance container | Pronti |
| **Ripeti per ogni container** *(task ripetuto)* | **Zero task ripetute** |

**Argomento contro repatriation**: chi torna on-prem deve mantenere internamente la subject matter expertise per ogni singolo servizio — costo nascosto enorme che le analisi di repatriation non considerano.

**Risorsa**: AWS Decision Guides (pagine ufficiali AWS per ogni categoria: compute, database, storage, networking) — meno del 20% dei professionisti AWS le conosce. Non sono le fonti primarie per l'esame (primary = Well-Architected white papers), ma sono supplemento utile.

---

### Principio: Mechanical Sympathy

**Significato:** scegliere il servizio/feature che corrisponde esattamente ai requisiti tecnici, non quello con cui si è più familiari.

**Area più difficile: Database** — le persone usano quello che conoscono meglio, non quello più adatto.

**Database Decision Tree:**

```
1. Shape: schema fisso?
   ├── Sì → Relational (RDS, Aurora)
   └── No → NoSQL
         2. Structure (se NoSQL): tipo di dato?
            ├── Key-Value → DynamoDB
            ├── Document → DocumentDB (MongoDB-compatible)
            ├── Graph → Neptune
            └── In-memory → ElastiCache, MemoryDB
         
3. Relational: scale e pattern di accesso?
   ├── Molti client distribuiti, cloud-native → Aurora
   └── Migrazione da on-prem, singolo client pesante → RDS
```

**Domande da rispondere prima di scegliere il database:**
1. Shape: NoSQL o Relational? (schema fisso?)
2. Structure: key-value, graph, document?
3. Size: quanti record? quanto pesa ogni record?
4. Read/Write ratio: quante letture e scritture per secondo?

---

### RDS vs Aurora — Differenze Chiave

| Aspetto | RDS | Aurora |
|---|---|---|
| Tipo | "Platform as a Service" (AWS gestisce OS e software) | Cloud-native, re-architettato |
| Target | Migrazione lift-and-shift da on-prem | Workload cloud-native, molti client distribuiti |
| Pricing | Meno costoso a scala piccola | Più costoso per workload piccoli o singoli client pesanti |
| Engine | MySQL, PostgreSQL, Oracle, SQL Server, MariaDB | MySQL-compatible, PostgreSQL-compatible |
| Multi-AZ | Standby in altra AZ (failover automatico) | Replicas in multiple AZ native |
| Accesso OS | ❌ (RDS Custom per casi speciali) | ❌ |

**Aurora Global Database** (active-passive multi-region):
- Una primary region + fino a 5 secondary read regions
- Failover: promuovi la secondary a primary → diventa cluster indipendente
- **Failback = processo manuale** (Aurora Global Database non ha failback nativo): bisogna creare una nuova cross-region replica nella direzione inversa, poi promuovere, poi riprovisioning — "not pretty"

**Aurora D-SQL** (active-active multi-region):
- Relational database disponibile attivo-attivo in multiple regioni simultaneamente
- Diverso da Global Database per use case (multi-region active-active vs. active-passive)

**Cross-Region RDS Read Replica:**
- Costo replicazione: ~$0.02/GB di dati trasferiti cross-region
- Failover = promozione della replica (diventa cluster indipendente)
- Failback = processo complesso (crea replica nella direzione inversa, promuovi, reprovisioning)

---

### Lambda — Ottimizzazione Performance

| Configurazione | Effetto |
|---|---|
| **Memory** | Aumenta proporzionalmente la vCPU; da 128MB a 10GB (fino a 6 vCPU) |
| **Node.js runtime** | Single-threaded → NON beneficia di più di 1 vCPU |
| **Provisioned Concurrency** | Pre-scalda istanze Lambda (elimina cold start) — migliore per latency, non per throughput |
| **Timeout più breve** | Riduce il costo delle invocazioni in error state (fail fast) |

🎯 **Trade-off**: se la funzione esegue già in 3ms, aumentare memory/vCPU non ha senso. Se il runtime è Node.js, i vCPU extra sono sprecati. Provisioned concurrency basso = costo basso per funzioni raramente invocate concorrentemente.

---

### Architettura Multi-Tier Serverless

```
Internet
    │
    ▼ DNS
Route 53 (globale, fault-tolerant, truly serverless)
    │
    ▼ CDN + cache
CloudFront (content caching, edge delivery, fault-tolerant)
    │
    ├── Static Content → S3 (serverless origin)
    │
    └── Dynamic Content
            │
            ▼
        API Gateway (front-end API)
            │
            ├── Cognito (autenticazione utenti)
            │
            ▼
        Lambda (business logic)
            │
            ▼
        DynamoDB (persistenza, serverless NoSQL)
```

**Note:**
- Lambda non è "blazingly fast" per latency — questa è una limitazione da riconoscere
- DynamoDB è key-value/document; per query SQL-like usa PartiQL (P-A-R-T-I-Q-L)
- DynamoDB Transactions supporta strong consistency per operazioni che ne hanno bisogno

**Architettura VPC-based vs Serverless:** spesso le aziende iniziano con EC2/VPC e migrano gradualmente verso serverless. Non è necessario (né realistico) essere cloud-native dall'inizio.

---

### IoT Core + EventBridge Pattern

```
IoT Device → AWS IoT Core → EventBridge Rule → Lambda (processing)
```

Questo pattern è valido quando i messaggi IoT devono scatenare automazione event-driven. EventBridge funge da router tra IoT Core e i servizi downstream.

---

## Cost Optimization Pillar — Domain 4/5 (Completo)

### Cloud Financial Management (FinOps) — Principi

Chad dedica ampio spazio a questo argomento, particolarmente rilevante per chi lavora in aziende enterprise.

**Quattro requisiti fondamentali:**

1. **Executive ownership**: deve esserci un functional owner del cloud financial management a livello VP/C-level. Deve poter parlare intelligentemente del cloud spend con i peer senza dover chiamare un tecnico. Se l'azienda non ha questo, la cost optimization è paralizzata.

2. **Finance-Technology partnership**: il team finance non può restare ignorante del cloud pricing. È loro responsabilità imparare come funziona. (Esempio reale: una società ha assunto una persona con background AWS e finance come liaison tra finance, technology e il C-level.)

3. **No sticker shock**: la comprensione del cloud pricing deve far parte del DNA dell'organizzazione. Budget e forecast adeguati permettono di identificare anomalie prima che diventino problemi.

4. **Cost-aware processes**: il costo deve essere integrato nei processi quotidiani (es. campo in JIRA per indicare il costo previsto di ogni ticket). Questo consente attribuzione del costo e accountability.

---

### EC2 Pricing Models

| Modello | Sconto vs On-demand | Volatilità | Caso d'uso |
|---|---|---|---|
| **On-demand** | 0% (baseline) | Nessuna | Workload irregolari, sviluppo, test |
| **Spot Instances** | Fino a 90% | Alta (terminate se spot price > bid) | Batch processing, workload fault-tolerant, CI/CD |
| **Reserved Instances** | Significativo | Nessuna (commitment 1 o 3 anni) | Baseline stabile e prevedibile |
| **Savings Plans** | Simile a RI | Nessuna | Come RI ma più flessibili (non legati a tipo istanza) |

**Spot Instances**: il prezzo di mercato varia. Se supera il bid, l'istanza viene terminata con 2 minuti di preavviso. Perfette per workload che tollerano interruzioni.

---

### Ottimizzazione Costi Ambienti Non-Produzione

**Scenario**: applicazione multi-tier con alta domanda nei giorni lavorativi, bassa il weekend.

**Approccio ottimale per cost + low operational effort:**

| Tier | Soluzione Non-Prod |
|---|---|
| Web proxy (EC2) | Spot Instances in ASG |
| Application runtime (EC2) | Spot Instances in ASG |
| Database (RDS) | **NON Spot** — RDS non supporta Spot; usare scheduled scaling (scale down after hours) |

🎯 **Chiave**: RDS non ha spot instances. Per risparmiare su RDS in non-prod: scale down automatico con scheduled actions. Per il compute: spot instances.

---

### Cost Data Sources (in ordine di granularità)

| Fonte | Granularità | Note |
|---|---|---|
| **PDF Bill mensile** | Bassa | Minima granularità |
| **Monthly CSV (se abilitato)** | Media | Feature da abilitare nelle billing settings; viene depositata in S3 |
| **Cost Explorer** | Buona | Visualizzazione interattiva, trend, forecasting |
| **AWS Budgets** | Buona | Avvisi su soglie di spesa e forecast |
| **Cost and Usage Report (CUR)** | **Massima** | Resource-level, depositato in S3, queryabile con Athena, visualizzabile con QuickSight |

🎯 **Insight**: se non abiliti il CSV mensile, quando perdi la history (es. migrazione di account tra organizzazioni) non puoi recuperarla. Il PDF è l'unica alternativa. CUR è la fonte più ricca ma non sempre disponibile storicamente.

**CUR + Athena + QuickSight** = stack standard per cloud cost analytics enterprise.

---

## Q&A Notevoli — Day 2

**Q: Con l'immutabilità (blue-green), si sostituisce l'intera stack?**  
**A:** Dipende dai requisiti. Blue-green può essere fatto a livello DNS, load balancer, target group, auto scaling group o anche combinazioni. È limitato solo dall'immaginazione e dai requisiti. Una singola LB con due target group è un approccio comune che evita di duplicare tutta l'infrastruttura.

**Q: Blue-green è meglio per app stateless?**  
**A:** Sì. Con LB stickiness (app stateful), un blue-green a livello DNS crea esperienza utente pessima perché le sessioni sono legate a una delle due infrastrutture.

**Q: API Gateway implica AWS API Gateway o potrebbe essere Apigee?**  
**A:** Nel contesto SAA-C03, è quasi sempre AWS API Gateway. Il terzo party come Apigee (ora parte di GCP) può apparire come "flavor" del problema ma raramente come soluzione. L'esame spinge a conoscere l'ecosistema AWS.

**Q: Il costo di replicazione del database cross-region è significativo?**  
**A:** Dipende dal rate of change del database. Il costo è ~$0.02/GB di dati trasferiti cross-region. Calcola quanto cambiano le tabelle nel tempo per stimare.

**Q: C'è un community per discutere le domande dopo il corso?**  
**A:** Chad ha provato Slack ma poca partecipazione. Meglio connettersi su LinkedIn direttamente.

**Q: Ha senso aumentare memory Lambda se esegue già in 3ms?**  
**A:** No. E se il runtime è Node.js, i vCPU extra non vengono usati (Node.js è single-threaded). Provisioned concurrency a numero basso è la scelta giusta per latenza, non per vCPU.

**Q: Throttling basato su latenza backend — esiste un servizio AWS nativo?**  
**A:** No. ALB non può rigettare traffico (solo routing). CloudFront può solo fare geoblocking. WAF rate-based usa soglie statiche. API Gateway usage plan usa soglie statiche. La soluzione è custom: Lambda su schedule che valuta la metrica di latenza e aggiorna dinamicamente la WAF rate-based rule.

---

## Takeaway Applicabili — Day 2

1. **Blue-green deployment livello LB/target group > livello DNS**: evita dipendenza dai TTL DNS; più controllo granulare.

2. **Immutabilità NON migliora performance**: utile per security e operational simplicity, non per speed.

3. **ECS Fargate = zero repetitive tasks** rispetto a EC2+Docker: il costo aggiuntivo è ampiamente giustificato dall'eliminazione di overhead operativo.

4. **Database selection**: rispondere a shape → structure → size → read/write ratio PRIMA di scegliere il database engine.

5. **Aurora Global Database: failback è un processo manuale complesso** — se serve failback automatico, considerare Aurora D-SQL (active-active) o soluzioni Route 53 ARC.

6. **CUR su S3 + Athena** è lo stack standard per analisi granulare dei costi — abilitarlo da subito, non quando serve.

7. **RDS in non-prod non supporta Spot**: usare scheduled scaling per risparmiare dopo orario lavorativo.

8. **Executive ownership del cloud spend è non-negoziabile** per una cost optimization efficace — senza sponsorship C-level, ogni iniziativa di cost management è condannata a fallire.

9. **Config rule + SSM remediation** per compliance automatica — funziona solo per cambiamenti booleani.

10. **No custom throttling nativo in AWS basato su latenza**: è un gap — la soluzione richiede Lambda + WAF.

---

## Materiali Aggiuntivi

- [ ] AWS Decision Guides (pagine ufficiali per ogni categoria di servizio)
- [ ] Well-Architected Framework white papers (Cost Optimization + Performance Efficiency pillars)
- [ ] Chad Smith su O'Reilly: AWS AI Practitioner corso (con demo live)



#fffcfc