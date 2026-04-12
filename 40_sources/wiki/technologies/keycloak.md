---
title: Keycloak Authorization
type: technology
tags: [thread6-security]
sources:
  - "[[Keycloak Authorization Concept]]"
  - "[[keycloak authorization concept blog post]]"
updated: 2026-04-09
related:
  - "[[concepts/independent-deployability]]"
---

# Keycloak — Fine-Grained Authorization

## Architettura: 5 building blocks

Keycloak Authorization Services implementa UMA 2.0 / OAuth2 con un policy engine:

| Concetto | Descrizione |
|---|---|
| **Resource Server** | Un Keycloak client che "possiede" risorse protette (es. `realm-management`) |
| **Resource** | Ciò che si vuole proteggere (es. un gruppo, un endpoint) |
| **Scope** | Un'azione sulla resource (`view`, `delete`, `manage-members`) |
| **Policy** | Una regola che valuta a PERMIT/DENY (es. "l'utente è nel gruppo X") |
| **Permission** | Lega Resource + Scope + Policy — la regola di accesso finale |

> **Insight chiave**: Policy e Permission sono **separate**. Una Policy risponde solo "questo utente passa?". Una Permission decide *dove* quella risposta viene applicata (Resource + Scope specifici).

## Policy Types

| Tipo | Classe Java | Uso |
|---|---|---|
| Role | `RolePolicyRepresentation` | Utente ha un ruolo specifico |
| Group | `GroupPolicyRepresentation` | Utente appartiene a un gruppo |
| User | `UserPolicyRepresentation` | Utente specifico |
| Time | `TimePolicyRepresentation` | Finestra temporale |
| JavaScript | `JSPolicyRepresentation` | Logica custom via JS |
| Aggregate | `AggregatePolicyRepresentation` | AND/OR di altre policy |

## Flusso di creazione (SPI)

```java
// 1. Trova il ResourceServer
ClientModel client = realm.getClientByClientId("realm-management");
ResourceServer rs = storeFactory.getResourceServerStore().findByClient(client);

// 2. Costruisci la Policy DTO
GroupPolicyRepresentation policyRep = new GroupPolicyRepresentation();
policyRep.setName("isFleetManagerOfAcme");
policyRep.setGroups(Set.of(new GroupDefinition(group.getId(), "/FleetManagers/Acme", false)));

// 3. Persisti la Policy
Policy policy = policyStore.create(rs, policyRep);
RepresentationToModel.toModel(policyRep, authzProvider, policy);

// 4. CRITICO: abilita fine-grained permissions sulla risorsa target
//    (crea automaticamente Resource + ScopePermissions)
mgmt.groups().setPermissionsEnabled(targetGroup, true);

// 5. Recupera le Permission auto-create
Map<String, String> permIds = mgmt.groups().getPermissions(targetGroup);

// 6. Collega la Policy alle Permission
policyStore.findById(realm, rs, permIds.get("manage-members"))
           .addAssociatedPolicy(policy);
```

## Gotcha critico

> `setPermissionsEnabled(group, true)` **non è solo un flag**. È la chiamata che crea effettivamente la Resource e le ScopePermission nel ResourceServer. Senza questa chiamata, non c'è nulla a cui collegare la policy.

**Ordine di cancellazione** (inverso alla creazione):
1. `removeAssociatedPolicy()` — stacca policy dalle permission
2. `policyStore.delete()` — cancella la policy
3. `setPermissionsEnabled(false)` — cleanup delle permission auto-create
4. `realm.removeGroup()` — infine rimuovi il gruppo

Invertire l'ordine lascia riferimenti pendenti o genera errori criptici.

## Classi SPI principali

```
org.keycloak.authorization
    ├── AuthorizationProvider     ← entry point (via session.getProvider())
    ├── model.Policy              ← domain object dopo la persistenza
    └── store
          ├── PolicyStore         ← CRUD policies
          ├── ResourceStore       ← CRUD resources
          └── ResourceServerStore ← trova ResourceServer da ClientModel

org.keycloak.models.utils
    ├── RepresentationToModel     ← DTO → model persistito
    └── ModelToRepresentation     ← model → DTO
```

## Connessioni

- [[concepts/independent-deployability]] — il Resource Server Keycloak è il gate di autorizzazione per i microservizi; in Microservices Fundamental Sam Newman descrive JWT come meccanismo standard per passare il contesto dell'utente autenticato tra servizi
