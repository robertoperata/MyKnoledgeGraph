---
tags:
  - architecture
  - microservices
feature: 
type: live event
author: "[[Sam Newman]]"
source: https://learning.oreilly.com/live-events/microservice-fundamentals/0636920054839/0642572189075/
---
# Microservices Fundamental

### Table of content:
- what are microservices?
- monoliths
- reasons to use microservices
- how to find boundaries
- ownership
- downsides of microservices
- DDD

## What are microservices?
>[!note] A type of service oriented architecture where the services are [[Independent deployability|independently deployable]] and boundaries are primarily defined by the business domain

key words are:
- *service oriented architecture*
- *[[Independent deployability|independently deployable]]*
- *business domain*

>[!question] How do we allow micro services to communicate with each other without becoming so coupled that they can't be changed independently?
- information hiding
- **communication is via some form of network call**:
	- some RPC framework (gRPC, SOAP)
	- rest
	- file

>[!quote] a lot of the challenges that we're going to look at from the world of microservices are  the challenges of any distributed system which is dealing with the network

One of the ways we make our microservices independently deployable is by reducing coupling.
One of the best way to reduce coupling is avoiding the use of shared database.

The styles of communication are
- **Request/Response**: when I need a piece of information
- **Event-Driven**: multiple services react to the same thing
usually will normally have a mix of these styles of communication
is common to expose interfaces that use both of these style

### Shared Libraries
If different version of libraries could work in different services is fine. otherwise not because against the *independently deployability*.
If I need the same code for all microservices at the same time, than create a service.

Is acceptable to have code duplication in different services (not the same) instead of shared libraries that bring to coupling.

### Security
to do authorisation you're ideally like to have each service decide what behaviour is authorised. So you need to be able to provide and pass down the context of the authenticated user in a way that is safe and secure most commonly using JWT token been sent over some form of secure transit.


## Monolith
- ***single process monolith***:
- ***modular monolith***: code is broken into modules, each module packaged together into a single process
- ***3rd party monolith***: coulbe be on-prem softwqare or a SAAS product, you have limited to no ability to change the core system
- ***Distributed monolith***: high cost to change, larger-scoped deployments, more to go wrong, release co-ordination

The decision to go to microservices from monolith should be motivated answering the question:
>[!question] what I can't do toady with the current architecture?
>- not able to scale
>- large team on same codebase
>- coordination for single deploy
>- unfrequent large deploy

## Reasons to use microservices
- reduce release co-ordination
- improve release frequency
- enables full lifecycle ownership
- different technology (language, database)
- scaling
- security (principle of least privilege and defence in depth)
### Finding Boundaries (in Business Domain)
- makes alignment with organisation structure easier
- domain boundaries tend to be more stable
- can recombine functionality more easily to deliver new experiences
independent deployability must be done in a way that the change to a service don't break upstream consumers. ***backwards compatibility is key***
**Information hiding** is a concept introduced in 1971 that means "Hiding things which are most likely to change behind a stable boundary" to define boundary that could be updated to fit microservices architecture with "a default approach whereby everything is hidden unless explicitly needed"
To be aware of what should be shared adopt a ***consumer-first mindset***

## Ownership
with microservices we are pushing ourselves much more towards business domain rather than technical bounded services

>[!quote] Organizations which design systems (in the broad sense used here) are constrained to produce designs which are copies of the communication structures of these organizations.

the solution is **feature team** that owns a business piece domain

Threat modelling is a process of understanding what have we got that people might want to steal, who wants to steal it and how could they steal

## Downsides of microservices

- Cognitive overload:
	- tooling
	- configuration
	- discovery
	- routing
	- observability
	- datastore
	- orchestration & deployment
	- deployment: languages & container
	- policy: architectural & security compliance

- Cross-service testing is more involved
- monitoring is more complex (open telemetry)
	- monitor not only system metrics but also business metrics
- resiliency isn't free 
	- every single time you expect something from another service you're going to ask: "what happen if it doesn't work?", "How I'm going to handle it?"
- security:
	- larger surgace area to attack
	- more things to protect (patch, monitor...)
## Domain-Driven Design
modelling the business domain in your code
### ubiquitous language
a common language of code and business so technical people and "business people" can talk. Verbs, nouns 
to get this language needs to talk and listen. There's a techniques like event storming that helps to do that
### bounded context
different responsibility in the organisation
boundary within your domain which hides details that has responsibilities:
- organisational grouping
- hiding complexity
#todo search Martin Fowler's post about bounded context

### aggregate
collection of objects that you always want to manage as a single unit of transactionality

![[microservicefundamentalsv41744722870152.pdf]]