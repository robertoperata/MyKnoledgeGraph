---
tags:
  - ai-governance
  - cloud-security
  - policy-as-code
  - data-classification
  - iam
feature:
type: article
author: Dave Ward
source: https://www.infoq.com/articles/governing-ai-cloud-guide/
date: 2026-06-28
---

# Governing AI in the Cloud: a Practical Guide for Architects

## Sunto

Il problema centrale affrontato dall'articolo è la cosiddetta "Shadow AI": secondo una ricerca Microsoft, il 71% dei dipendenti ha utilizzato strumenti AI non approvati dall'azienda e il 51% lo fa settimanalmente. I modelli di sicurezza tradizionali assumono un'infrastruttura nota e controllata, ma l'AI rompe questa ipotesi con grande rapidità quando i team di sviluppo integrano API esterne su dati reali dei clienti senza seguire un processo formale di approvazione.

L'articolo propone tre approcci complementari per scoprire dove e come l'AI viene utilizzata all'interno dell'infrastruttura: Cloud Access Security Brokers (CASB) come Defender for Cloud Apps e Netskope, che identificano le chiamate verso provider come OpenAI, Anthropic e Hugging Face; il Service Mesh Telemetry in ambienti Kubernetes tramite Istio o Linkerd, che rivela l'utilizzo a livello di pod; e l'analisi degli audit log degli API Gateway (AWS API Gateway, Kong, Apigee), che cattura ogni pattern di chiamata esterna con dettaglio interrogabile.

La strategia di classificazione dei dati proposta prevede un'etichettatura obbligatoria al momento della creazione, molto più efficiente del processo retroattivo su dati già distribuiti. I tool suggeriti includono AWS Macie per la scoperta batch, Amazon Comprehend per il rilevamento in tempo reale via Lambda, Microsoft Purview per Azure e Google DLP per GCP. I tag standard raccomandati sono: `DataClassification` (Public/Internal/Confidential/Restricted), `ContainsPII`, `AIApproved`, `ScanDate` e `ComplianceScope`.

Il framework IAM proposto si struttura su cinque livelli di policy deny-based che impediscono l'accesso non autorizzato: blocco degli upload senza tag di classificazione, blocco delle letture senza classificazione, blocco se non esplicitamente approvato per uso AI, accesso consentito solo tramite ruoli IAM specifici via endpoint VPC, e divieto assoluto sui dati "Restricted". Questo modello utilizza statement `Deny` espliciti con `Principal "*"` per prevenire aggiramento tramite assunzione di ruoli.

L'articolo conclude sottolineando che la tecnologia è la parte semplice: la vera sfida è allineare security, engineering e product team con ownership chiare e workflow automatizzati che non dipendano da approvazioni manuali. La governance AI efficace si trasforma da checkpoint post-hoc a componente integrale della pipeline di delivery, con l'automazione che gestisce la validazione ripetitiva e gli esseri umani che si concentrano sulle decisioni complesse.

## Codice

Policy IAM con cinque livelli di deny-based enforcement per dati classificati:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUploadWithoutClassification",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::data-bucket/*",
      "Condition": {
        "Null": {
          "s3:RequestObjectTag/DataClassification": "true"
        }
      }
    },
    {
      "Sid": "DenyReadWithoutClassification",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::data-bucket/*",
      "Condition": {
        "Null": {
          "s3:ExistingObjectTag/DataClassification": "true"
        }
      }
    },
    {
      "Sid": "DenyReadIfNotAIApproved",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::data-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:ExistingObjectTag/AIApproved": "true"
        }
      }
    },
    {
      "Sid": "DenyRestrictedData",
      "Effect": "Deny",
      "Principal": "*",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::data-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:ExistingObjectTag/DataClassification": "Restricted"
        }
      }
    }
  ]
}
```

Wrapper SecureS3Client che astrae la complessità compliance per gli sviluppatori:

```python
from dataclasses import dataclass
from typing import Literal
import boto3

DataClassification = Literal["Public", "Internal", "Confidential", "Restricted"]

@dataclass
class SecureS3Client:
    bucket_name: str
    kms_key_id: str

    def upload_data(
        self,
        key: str,
        data: bytes,
        classification: DataClassification,
        contains_pii: bool = False,
        ai_approved: bool = False,
    ) -> None:
        if classification == "Restricted":
            raise ValueError("Restricted data cannot be uploaded for AI use")

        tags = {
            "DataClassification": classification,
            "ContainsPII": str(contains_pii).lower(),
            "AIApproved": str(ai_approved).lower(),
            "ScanDate": self._today(),
        }

        s3 = boto3.client("s3")
        s3.put_object(
            Bucket=self.bucket_name,
            Key=key,
            Body=data,
            ServerSideEncryption="aws:kms",
            SSEKMSKeyId=self.kms_key_id,
            Tagging="&".join(f"{k}={v}" for k, v in tags.items()),
        )
```

Policy Rego (OPA) per decisioni contestuali su modelli AI in produzione:

```rego
package ai.governance

default allow = false

allow {
    input.data_classification != "Restricted"
    model_is_registered(input.model_id)
    security_scan_is_fresh(input.model_id)
    not data_retention_exceeded(input.data_created_at)
}

model_is_registered(model_id) {
    data.model_registry[model_id].status == "approved"
}

security_scan_is_fresh(model_id) {
    scan_date := data.model_registry[model_id].last_security_scan
    days_since_scan := (time.now_ns() - scan_date) / (24 * 60 * 60 * 1000000000)
    days_since_scan <= 7
}

data_retention_exceeded(created_at) {
    input.data_classification == "Customer"
    age_days := (time.now_ns() - created_at) / (24 * 60 * 60 * 1000000000)
    age_days > 90
}

deny[reason] {
    not model_is_registered(input.model_id)
    reason := sprintf("Model %v is not registered in the approved registry", [input.model_id])
}

deny[reason] {
    not security_scan_is_fresh(input.model_id)
    reason := sprintf("Security scan for model %v is older than 7 days", [input.model_id])
}
```
