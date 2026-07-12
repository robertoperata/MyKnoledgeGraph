---
tags:
  - ai-assisted-development
  - software-architecture
  - design-patterns
  - code-quality
  - template-method
feature:
type: article
author: Patrick Farry
source: https://www.infoq.com/articles/skeleton-architecture/
date: 2026-07-12
---

# The Skeleton Architecture: How to Grow a Working System from Day One

## Sunto

L'articolo di Patrick Farry affronta il problema della qualità del codice e del debito tecnico nell'era degli assistenti AI. L'autore sostiene che la struttura e i guardrail architetturali — non solo i prompt — siano essenziali per uno sviluppo AI-assistito sicuro e di qualità. Il problema centrale è la limitazione della finestra di contesto dei modelli AI: all'aumentare del contesto, l'accuratezza diminuisce per il fenomeno "lost in the middle", mentre latenza e costi aumentano. La soluzione passa per la minimizzazione dello scope che il modello deve mantenere in memoria.

Farry analizza due pattern architetturali per il coding AI-assistito. L'**Atomic Architecture** organizza i sistemi da atomi irriducibili verso l'alto (utility functions che si combinano in molecole e organismi), garantendo codice generato isolato e focalizzato, ma creando overhead significativo nel collegare componenti disparati. La **Vertical Slice Architecture** organizza i sistemi per feature di business anziché per layer tecnici, mantenendo insieme le modifiche correlate e riducendo la contaminazione del contesto, a rischio però di duplicazione del codice tra le slice.

Il contributo originale dell'autore è il modello **Scheletro e Tessuto**: i sistemi vengono separati in due domini distinti. Lo **Scheletro** comprende strutture definite dall'uomo e immutabili (classi astratte, interfacce, security context) che impongono i requisiti non funzionali. Il **Tessuto** è costituito da feature ad alto contenuto implementativo generate dall'AI (classi concrete, business logic). Questo approccio sfrutta il pattern Template Method: l'architetto umano definisce un metodo `run()` final che gestisce le preoccupazioni trasversali, mentre l'AI implementa soltanto il metodo `_execute()` protetto.

Per rafforzare i guardrail, l'architettura si avvale di diversi meccanismi: validazione JSON Schema e OpenAPI/AsyncAPI per imporre i contratti dei dati, validatori fail-fast che causano crash immediato in caso di violazioni, strumenti CI/CD come ArchUnit per prevenire scorciatoie architetturali, repository read-only per separare le definizioni dello scheletro su cui l'AI non ha permessi di modifica, e isolamento degli effetti collaterali nello scheletro con business logic pura e testabile nel tessuto.

L'articolo conclude con un importante cambiamento di prospettiva professionale: gli sviluppatori devono spostare il focus dalla sintassi al systems thinking, concentrandosi sulla decomposizione dei problemi, il modellamento dei flussi informativi e la gestione dei requisiti non funzionali. Lo skeleton architecture viene proposto anche come framework di apprendistato per sviluppatori junior con AI, sostituendo "il paralizzante problema della pagina bianca" con un esercizio strutturato di completamento in cui il feedback immediato (crash del sistema contro i guardrail) insegna i principi architetturali più velocemente dei cicli tradizionali di code review.

## Codice

La classe base `BaseTask` dove lo scheletro gestisce buffering e verifica di disponibilità, mentre l'AI implementa solo il metodo `process()`:

```python
class BaseTask(ABC):
    def final run(self):
        # skeleton handles buffering, readiness, crosscutting
        while not self._is_ready():
            time.sleep(0.1)
        result = self._execute()
        self._buffer(result)
        return result
    
    @abstractmethod
    def _execute(self):
        """AI implements only this method"""
        pass
```

Un validatore `MQTTValidator` che impone l'integrità dello schema con `sys.exit(1)` in caso di violazioni — un hard constraint che l'AI non può aggirare:

```python
class MQTTValidator:
    def validate(self, message):
        try:
            jsonschema.validate(message, self.schema)
        except jsonschema.ValidationError as e:
            logging.error(f"Schema violation: {e.message}")
            sys.exit(1)  # fail-fast: hard constraint, non bypassabile
```
