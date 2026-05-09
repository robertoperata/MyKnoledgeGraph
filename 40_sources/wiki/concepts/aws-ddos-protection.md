---
title: AWS DDoS Protection e Security Architecture
type: concept
tags: [thread3, aws]
sources: 
  - "[[courses/hands-on-with-aws-vpcs/knowledge]]"
  - "[[courses/aws-security-deep-dive-vpcs/knowledge]]"
updated: 2026-05-03
related:
  - "[[concepts/aws-vpc-fundamentals]]"
  - "[[concepts/aws-security-groups-nacls]"
  - "[[concepts/aws-vpc-connectivity]]"
---

# AWS DDoS Protection e Security Architecture

## Definizione

L'insieme di servizi e pattern AWS per proteggere workload da attacchi DDoS e accessi non autorizzati. Il principio fondamentale è **defense in depth**: più layer indipendenti, ognuno gestito da un servizio managed AWS.

## Principio chiave: obfuscation e managed services

> *"Place your vulnerable systems behind AWS managed services."* (AWS Well-Architected Framework, Security Pillar)

Gli attaccanti devono attraversare AWS (CloudFront, Route 53, ALB) prima di raggiungere le risorse che gestisci tu. AWS ha capacità di assorbimento enormemente superiori a qualsiasi singolo cliente.

Analogia (Rick Crisci): il castello medievale vuole fermarti al fossato — prima che tu arrivi alle mura, prima che arrivi al portone. Ogni layer che superi è più vicino al danno.

## Architettura a layer

```
Internet
    │
[Route 53] ← Shield Standard (built-in) + DNS-level protection
    │
[CloudFront] ← AWS WAF + Shield + caching edge
    │
[Application Load Balancer] ← AWS WAF + Security Groups
    │
[EC2 Web Servers] ← Security Groups + NACLs + Auto Scaling
```

Ogni layer che il traffico deve attraversare è un'opportunità di blocco. La maggior parte degli attacchi viene fermata prima di raggiungere le istanze EC2.

## AWS Shield

**Shield Standard (gratuito)**:
- Automaticamente incluso per tutti i clienti AWS
- Protegge Route 53, CloudFront, ALB, ELB
- Protegge contro attacchi layer 3/4: SYN floods, UDP floods, reflection attacks
- Route 53 è così scalabile che è praticamente immune a DDoS volumetrici

**Shield Advanced (a pagamento)**:
- Protezione avanzata per EC2, ELB, CloudFront, Route 53, Global Accelerator
- Supporto del DDoS Response Team (DRT) di AWS
- Copertura dei costi AWS aggiuntivi durante un attacco
- Dashboard e metriche dettagliate sugli attacchi in corso

## AWS WAF (Web Application Firewall)

Firewall layer 7 gestito da AWS, integrato in CloudFront, ALB, API Gateway.

**Modalità di configurazione:**
1. **Managed Rules**: AWS configura e aggiorna le regole automaticamente — zero effort
2. **Custom Rules**: condizioni specifiche (IP block list, geo-blocking, SQL injection, XSS ecc.)
3. **Rate-based Rules**: de-prioritizza sorgenti con troppe richieste in un intervallo di tempo

**Rate-based rules contro DDoS layer 7:**
- WAF osserva il traffico, identifica sorgenti ad alto volume
- De-prioritizza le richieste delle sorgenti attaccanti
- Il traffico legittimo passa normalmente
- Il blocco avviene "al bordo" (CloudFront edge), mai vicino ai web server

**WAF Sandwich** (pattern con firewall di terze parti):
```
Internet
    │
[NLB internet-facing]
    │
[EC2 firewall: Palo Alto / Fortinet / Checkpoint / Barracuda]
    │ (auto scaling group)
[Internal ALB]
    │
[Web Server (no public IP)]
```
Utile quando si vuole usare il proprio firewall on-prem preferito, o si ha bisogno di full control sul firewall. Il firewall EC2 deve essere gestito (patching, HA).

## Route 53 e CloudFront come prima linea

**Route 53:**
- DNS managed + Shield Standard built-in
- Così massivamente scalabile da essere immune a DDoS volumetrici
- Protegge contro HTTP floods prima che il traffico entri in CloudFront

**CloudFront:**
- CDN con edge location globali — risponde alle richieste HTTP/HTTPS direttamente dall'edge
- Assorbe HTTP floods, SYN floods, TLS request floods senza toccare i web server
- Nasconde il vero IP dei web server (obfuscation)
- AWS WAF integrato filtra all'edge

Traffico tipico: il 70-80% delle richieste viene servito dalla cache CloudFront senza mai raggiungere il web server.

## Auto Scaling come mitigazione DDoS

**Approccio**: invece di bloccare il DDoS, si scala orizzontalmente per assorbire il traffico.

**Logica economica (Rick Crisci)**:
```
1.000 EC2 T3.large × $0.08/ora × 24 ore = $1.920
```
Se il costo del downtime supera $2K, l'auto scaling vale. Gli attacchi DDoS non riusciti durano da pochi minuti a poche ore — se non funziona, l'attaccante si sposta su un target più facile.

**Critica comune**: "la bolletta esploderà." Ma il costo del downtime è solitamente superiore al costo delle istanze.

## Tipi di attacchi e layer di difesa

| Tipo attacco | Dove viene fermato |
|---|---|
| Volumetrico (DDoS layer 3/4) | Route 53 + Shield Standard |
| HTTP flood (layer 7) | CloudFront + AWS WAF |
| SYN flood, TLS overload | CloudFront |
| Application-level (SQLi, XSS) | AWS WAF su ALB |
| Traffico anomalo che passa | Auto Scaling + EC2 Security Groups |

## AWS Macie e GuardDuty (threat detection)

**AWS Macie** (ML-driven):
- Scopre automaticamente dati sensibili in S3 (PII, credenziali, dati finanziari)
- Alerting automatico in caso di accessi anomali
- Non è DDoS protection ma protezione dati sensibili

**AWS GuardDuty** (threat detection intelligente):
- Monitora continuamente account, workload, traffico VPC Flow Logs
- Usa ML per identificare pattern anomali (crypto mining, C2 communication, privilege escalation)
- Rileva credenziali compromesse, comportamenti insoliti
- Si integra con Shield e WAF per risposta automatizzata

## IaC per sicurezza

Definire la VPC via CloudFormation o Terraform garantisce:
- Configurazioni di sicurezza riproducibili (SG, NACL, WAF rules)
- Dev/prod parity — stesso template, parametri diversi
- Rollback rapido a configurazione precedente in caso di errore

## Trade-off

| Approccio | Pro | Contro |
|---|---|---|
| Shield Standard (gratuito) | Zero costo, automatico | Solo layer 3/4 di base |
| Shield Advanced | Protezione avanzata, DRT support | Costo mensile significativo |
| WAF Managed Rules | Zero configurazione | Meno controllo granulare |
| WAF Custom Rules | Controllo preciso | Expertise + manutenzione |
| WAF Sandwich EC2 | Firewall preferito, full control | Gestione OS, HA a carico tuo |
| Auto Scaling anti-DDoS | Sito resta up | Costo AWS più alto durante attacco |
| CloudFront | Blocco all'edge, CDN | Latency aggiuntiva per contenuto dinamico |

## Connessioni con altri concetti

- [[concepts/aws-vpc-fundamentals]] — la VPC e i SG sono l'ultimo layer di difesa
- [[concepts/aws-security-groups-nacls]] — SG e NACL si integrano nell'architettura defense-in-depth
- [[concepts/aws-vpc-connectivity]] — Direct Connect riduce la superficie di attacco eliminando il traffico da internet
