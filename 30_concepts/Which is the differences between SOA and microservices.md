---
tags: 
feature: 
type: 
author: 
source: 
---
# Which is the differences between SOA and microservices

Great question! **[[Service Oriented Architecture (SOA)|SOA (Service-Oriented Architecture)]]** and **[[Microservices Architecture]]** are closely related — both are service-based, but they differ in **scope, granularity, and design philosophy**.

Here's a clear breakdown:

|Feature|**SOA (Service-Oriented Architecture)**|**Microservices Architecture**|
|---|---|---|
|**Granularity**|Coarse-grained (services are larger and more general)|Fine-grained (services are smaller and more focused)|
|**Communication**|Often uses **enterprise protocols** like **SOAP**, XML, via ESB|Prefers **lightweight protocols** like **REST**, gRPC, JSON|
|**Shared resources**|Typically uses a **shared database** across services|Each service has **its own database** (database per service pattern)|
|**Service independence**|Services are reusable but more **tightly coupled** (often through ESB)|Services are **fully independent**, deployable, and loosely coupled|
|**Technology stack**|Often standardized across the enterprise|Can use **different tech stacks** per service|
|**Governance**|Centralized governance and security policies|Decentralized governance; teams own their services|
|**Use of ESB (Enterprise Service Bus)**|Common — used for integration, routing, transformation|Avoided — microservices prefer **smart endpoints and dumb pipes**|
|**Deployment**|Often deployed as a **monolithic suite** or in parts|Services are **independently deployable** and scalable|
|**Example Use Case**|Large enterprise systems (e.g., ERP, legacy modernization)|Agile, cloud-native apps (e.g., e-commerce, SaaS platforms)|

### Summary:

- **SOA** is like building a city with shared infrastructure (e.g., central plumbing, power lines).
    
- **Microservices** is like building many self-sufficient tiny homes — each with its own utilities and independence.
    

Would you like a visual diagram or recommendation on when to use one over the other?