---
tags: 
feature: 
type: 
author: 
source: 
---
# twelve-factor application

## 1. Codebase
single codebase for the application
git for each project

## 2. Explicitly declare and isolate dependency
>[!quote] Never relies on implicit existence of system-wide packages, 

docker to package to a container

## 3. Config
>[!quote] the twelve-factor app stores config in environment variables

## 4. Backing services
>[!quote] treat backing services as attached resources


## 5. Build, release, run
>[!quote] The twelve-factor app uses strict separation between the build, release, and run stages

## 4. Processes
>[!quote] twelve-factor processes are stateless and share-noting

>[!quote] sticky sessions area a violation of twelve-factor and hould never be used or relid upon.

## 7. Port Binding
>[!quote] The twelve-factor app is completely self-contained

## 8. Concurrency
applications should scale horizontally

## 9. Disposability
>[!quote] the twelve-factor ap's processes are disposable, meaning they can be started or stopped at a moment's notice

>[!quote] the twelve-factor app;s processes should shutdown gracefully when receive a SIGTERM signal from the process manager

## 10. Dev/prod parity
>[!quote] The twelve-factor app is designed for continuous deployment by keeping the gap between development and production small

>[!quote] The twelve-factor develper resists the urge to use different backing services between development and production

## 11. Logs
>[!quote] A twelve-factor app never concerns itself with routing or storage of its output stream

>[!quote] Store logs in a centralised location in a structured format

should uses an agent to transfer to a centralised location
fluentd, elk, splunk

## 12. Admin processes
>[!quote] Administration tasks should be kept separated from the application process, and should be run in an identical setup and  be automated, scalable and reproducible

