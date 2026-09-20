---
tags:
  - mtls
  - zero-trust
  - ci-cd
  - pomerium
  - sicurezza
feature:
type: article
author: Nick Taylor
source: https://labs.iximiuz.com/tutorials/secure-machine-to-machine-access-with-mtls-and-pomerium-2fce5d89
date: 2026-09-20
---

# Secure Machine-to-Machine Access with mTLS and Pomerium

## Sunto

Questo tutorial affronta un problema critico nella sicurezza dei pipeline CI/CD: i workload headless (runner CI, pipeline CD, batch job) devono accedere ad API interne senza credenziali statiche tipo API key o token di servizio. Tali credenziali sono "bearer token" — chiunque le ottenga può usarle finché rimangono valide. La soluzione proposta è **mutual TLS (mTLS)** combinato con Pomerium per l'application-level authorization.

La distinzione fondamentale del pattern è questa: **mTLS prova il possesso della chiave privata client durante il TLS handshake senza trasmetterla**, ma questo non decide a cosa la macchina può accedere. Pomerium aggiunge il layer di autorizzazione: il proxy si fida di una CA in modo ampio, ma autorizza solo runner specifici tramite certificate fingerprint. La separazione tra autenticazione (chi sei) e autorizzazione (a cosa puoi accedere) è il principio architetturale centrale.

L'architettura prevede due nodi: il **runner** (che tiene il certificato client e la chiave privata, si fida della CA del server) e il **gateway** (che esegue Pomerium con certificato server, si fida della CA client, valida i certificati e li autorizza per fingerprint prima di fare proxy verso l'API interna). La policy di Pomerium usa un deny esplicito per certificati invalidi e un allow per fingerprint specifici, permettendo hot-reload della configurazione senza riavvio.

Una capacità critica del sistema è la **rotazione dei certificati a zero downtime**: si genera una nuova coppia chiave/certificato, si autorizzano entrambi i fingerprint (vecchio e nuovo) durante il periodo di overlap, si fa la migrazione e si rimuove il vecchio fingerprint. Questo è impossibile con token statici. Pomerium ricarica la configurazione senza restart, quindi la revoca è istantanea.

Per la produzione, il tutorial consiglia di spostare le chiavi di firma della CA in Hardware Security Module (HSM), implementare sistemi automatizzati di emissione e rotazione dei certificati, usare meccanismi di workload identity quando disponibili (GitHub OIDC, service mesh), configurare CRL per revoca time-sensitive, e monitorare la scadenza dei certificati e i fallimenti di autorizzazione.

## Codice

Configurazione Pomerium per mTLS con fingerprint authorization:

```yaml
downstream_mtls:
  ca_file: /etc/pomerium/certs/client-ca-cert.pem
  enforcement: policy

routes:
  - from: https://internal-api.example.com
    to: http://localhost:8080
    policy:
      - deny:
          and:
            - invalid_client_certificate: true
      - allow:
          and:
            - client_certificate:
                fingerprint: 'SHA256:abc123...'
```

Esempio di chiamata curl con certificato client verso l'API protetta:

```bash
# Chiamata autenticata con certificato client
curl --cert /etc/runner/client.crt \
     --key /etc/runner/client.key \
     --cacert /etc/runner/server-ca.crt \
     https://internal-api.example.com/endpoint

# Risposta per certificato valido ma fingerprint non autorizzato: HTTP 403
# Risposta per CA non corrispondente: HTTP 495 (SSL Certificate Error)
# Risposta per fingerprint autorizzato: HTTP 200
```

Workflow GitHub Actions con mTLS (pattern generico):

```yaml
jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - name: Call internal API
        run: |
          curl --cert ${{ secrets.CLIENT_CERT_PATH }} \
               --key ${{ secrets.CLIENT_KEY_PATH }} \
               https://internal-api.example.com/deploy
```
