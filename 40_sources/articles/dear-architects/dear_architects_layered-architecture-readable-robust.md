---
tags:
  - layered-architecture
  - software-design
  - clean-architecture
  - python
  - separation-of-concerns
feature:
type: article
author: Mike Huls
source: https://mikehuls.com/layered-architecture-for-building-readable-robust-and-extensible-apps/
date: 2026-09-13
---

# Layered Architecture for Building Readable, Robust, and Extensible Apps

## Sunto

L'articolo presenta un approccio pratico all'organizzazione del codice attraverso un'architettura a layer con responsabilità chiaramente separate. Il concetto guida è che "good structure doesn't add ceremony. It preserves momentum": un'architettura ben strutturata riduce il carico cognitivo, semplifica il testing e rende i cambiamenti più sicuri man mano che la complessità del sistema cresce.

Il modello proposto si articola in **cinque layer**: il **Domain Layer** contiene i modelli di dati (dataclass o Pydantic) con vincoli e significato business, comprensibile anche da stakeholder non tecnici; l'**Application Layer** orchestra la business logic e i workflow coordinando domain model, repository e servizi di infrastruttura — se questo layer può essere testato senza database o web server, l'architettura è corretta; l'**Infrastructure Layer** fornisce strumenti (logging, hashing, HTTP client, message queue, email) senza contenere business logic; l'**Interface Layer** funge da gateway verso sistemi esterni con responsabilità minima (ricevere input, validare, chiamare l'application layer, formattare la risposta); il **Repository Layer** definisce i confini di persistenza disaccoppiando la business logic dall'implementazione del database.

La regola fondamentale è che le dipendenze fluiscono verso l'interno: la business logic rimane indipendente da framework, database e meccanismi di trasporto. Questa isolazione migliora la chiarezza architetturale e semplifica drasticamente il testing — è possibile testare l'intera logica di business con semplici unit test, senza spin-up di infrastruttura.

Per applicazioni di maggiori dimensioni, l'autore raccomanda di combinare il layering orizzontale con il **vertical slicing per dominio**: ogni dominio (es. `subscriptions`, `users`) ha la propria struttura a layer interna, con tre regole di separazione — i domini non importano gli internals degli altri (usano DTO o interfacce esplicite), i concetti condivisi appartengono a un dominio `shared` che cambia lentamente e contiene astrazioni piuttosto che workflow, e le dipendenze fluiscono verso l'interno anche all'interno di ogni slice.

I benefici pratici sono concreti: endpoint refactorizzati diventano testabili senza web server o database, i componenti di infrastruttura sono rimpiazzabili (cambio di ORM, di message broker, di provider email) senza toccare la business logic, il carico cognitivo si riduce perché ogni layer ha una responsabilità chiara, e i cambiamenti avvengono per estensione anziché per riscrittura. Man mano che la complessità aumenta, il costo del cambiamento rimane basso invece di crescere esponenzialmente.

## Codice

Struttura del progetto con vertical slicing per dominio:

```
src/
  subscriptions/
    domain/       # Modelli Pydantic, vincoli business
    application/  # Orchestrazione workflow, business logic
    repository/   # Accesso al database, ORM models
    infrastructure/ # Strumenti: email, cache, ecc.
    interface/    # API endpoints, CLI, event handlers
  users/
    domain/
    application/
    repository/
    infrastructure/
    interface/
  shared/         # Astrazioni condivise (cambiano lentamente)
```

Flusso delle dipendenze nel Domain Layer (esempio con Pydantic):

```python
# domain/subscription.py
from pydantic import BaseModel, validator
from datetime import datetime

class Subscription(BaseModel):
    """Comprensibile anche a stakeholder non tecnici"""
    user_id: int
    plan: str
    started_at: datetime
    active: bool = True
    
    @validator('plan')
    def plan_must_be_valid(cls, v):
        if v not in ['basic', 'pro', 'enterprise']:
            raise ValueError(f'Piano non valido: {v}')
        return v
    
    # Solo vincoli e significato business
    # Nessuna dipendenza da database, framework o transport
```

Application Layer — testabile senza infrastruttura:

```python
# application/subscription_service.py
class SubscriptionService:
    def __init__(self, repo: SubscriptionRepository, email: EmailService):
        self.repo = repo
        self.email = email
    
    def activate_subscription(self, user_id: int, plan: str) -> Subscription:
        # Orchestrazione: validazione, persistenza, notifica
        subscription = Subscription(user_id=user_id, plan=plan)
        self.repo.save(subscription)
        self.email.send_confirmation(subscription)
        return subscription
    
# Test senza database o email server:
def test_activate_subscription():
    mock_repo = MockSubscriptionRepository()
    mock_email = MockEmailService()
    service = SubscriptionService(mock_repo, mock_email)
    result = service.activate_subscription(user_id=1, plan='pro')
    assert result.active == True
    assert mock_repo.saved_count == 1
```

Interface Layer — responsabilità minima:

```python
# interface/api.py
@router.post("/subscriptions")
async def create_subscription(request: CreateSubscriptionRequest):
    # 1. Ricevere input
    # 2. Validare/normalizzare
    subscription_data = validate_request(request)
    # 3. Delegare all'application layer
    result = subscription_service.activate_subscription(
        user_id=subscription_data.user_id,
        plan=subscription_data.plan
    )
    # 4. Formattare risposta
    return SubscriptionResponse.from_domain(result)
    
# Nessuna business logic qui — solo traduzione dal/verso l'esterno
```
