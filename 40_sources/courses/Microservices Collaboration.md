---
tags:
  - microservices
feature:
type: live event
author: "[[Sam Newman]]"
source: https://learning.oreilly.com/live-events/microservice-collaboration/0636920064353/
---
# Microservices Collaboration

Microservices *inherently a type of distributed system*
distributed = when 2 or more computers are communication each over on local network
SOA (another distributed system) that differ for independent deployability
boundaries primarily defined by the domain

Communication is via some form of network call
- udp/tcp
- http
- files
data is hidden inside the microservice boundary (database per microservice)
microservices benefits:
- reduce release co-ordination
- improve release frequency 
- enables full lifecycle ownership

>[!question] Api gateway vs network gateway

## Finding Boundaries
the goal is to find boundaries to reduce coupling between services and that allow to be deployed independently
- how do we get good at maintaining backwards compatibility?
	- hiding most possible things and allow access to few public api, must be explicit about what is shared and what is hidden
>[!tip] Hide information/behavior that changes behind a stable boundary

Microservices exists to be consumed (called) but also to provide functionality to other things (other microservices, external services, external clients, third part party API)

>[!tip] We want to thing about things from the point of view of the consumer. Think Outside-In

>[!tip] Treat your microservice endpoints like a user interface

#### -  The more you hide, the easier it is to maintain backwards compatibility.
#### - Don't expose anything until you need it.
#### - Be explicit

A consequence about having services which are explicitly defined service interfaces, is to have explicit schema (because you always have a schema)
#### - You always have a schema - but is it explicit or implicit?

>[!question] How to use openapi-diff to compare those schema version

microservices
indipendent deployable
information hiding
hiding data
inside out
schemas 

## Styles of communications
- request/response: the consumer request a piece of data. It's not a command because the server could not be able to do that. if able the expected answer could be "done it" (202) or a some structured response. The intent of the collaboration is from the serivce that sends the requests
- event-driven: something happen and interested parties receive event and react. The intent of the collaboration lives with the recipient.

>[!question] what if we are dealing with a request/response to one endpoint but the request triggers a long-running task in the microservice?
>- request 201 - polling
>- request 201 - response in different queue (or a topic?)

when we thing at compatibility and versioning of an end point with API, we look at the API version as a whole. If I have one version of my API and I need to change it in some way I might end up with having to have a full separate version of my microservice endpoint.
whereas in events I converge in each individual event in a different format 

>[!todo] Look Tolerant reader pattern
>describe where you should be tolerant to the payload you're being sent changed in small ways that you don't care about

### Call within a process boundary are more different than we might think

- distributed system are complex. That complexity comes from two fundamental constraints:
	1. you cannot beam information instantaneously between two points
	2. sometimes the things you want to talk to is not there

>[!quote] Distributed system: a computer you've never heard of stops your computer from working

the overhead of a method is very low (movement of data by reference), the overhead can be very high (data moved by marshalling of handing off to an external store)\

In a distributed system should handle timeouts, downstream outage, and a much richer set of error handling semantic as well.
4xx client errors (you did something wrong) - 28 http error code
	410 = the thing you ask for is not there anymore
5xx server errors () - 11 http error code
	503 - 504
## Synchronous vs Asynchronous
### blocking / non-blocking
client concern
future, reactive programming

if need result from calls or that something has been done:
`await` 

### Temporal (de)coupling
when to or more processes have to be up and available at the time for an operation to complete
**Intermediaries can be used to remove temporal coupling** (broker)

broker base request/response good for long running task
each instance of microservice that handle response must be able to handle in a statlessly way

#### Coordinating multiple services
2 phase commit
sagas
##### Sagas
key thing you need for a saga pattern
- execution log: a record of what has happened
- execution coordinator: something to tell you what to do
**Orchestration** (command in control)
Pros:
- explicit representation of business process
- know in-line if there has been a problem
Cons:
- can be fairly couple
- can lead to overly smart (and dumb services)
- problematic for large orgs

**Choreography** (hippi drum circle)
Pros:
- highly decoupled
- evenly distributed smarts
Cons:
- lost explicit business process mapping
- understanding completion or error state is complex


![[microservicecollaborationoreillytrainingv21700150945847.pdf]]