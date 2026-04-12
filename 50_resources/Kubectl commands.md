---
tags:
  - k8s
feature:
type:
author:
source:
---
# Kubectl commands

 - `kubectl expose` can be uses to create services providing access to Deployments, ReplicaSet, Pods and other services
 - `kubect create service --port` --port argument must be specified to indicate the port on which the service will be listening for the incoming traffic. usually equivalent to the application port in which the application is listening
 - `kubectl create deployment nginxsvc --image=nginx`
 - `kubectl scale deployment nginxsvc --replicas=3`
 - `kubectl expose deployment nginxsvc --port=80` create service
 - `kubectl get all --selector app=nginxsvc`
 - `kubectl get svc nginxsvc -o yaml` print the yaml of svc
 - `kubectl get endpoints` get all services with their endpoints
 - `kubectl label pod nginx success=true` add label
 - `kubectl label pod nginx success-` remove label
 - `kubect create ing -h | less` **show documentation**
 - `kubectl create ing nginxsvc-ingress --rule="/=nginxsvc:80"`
 - `kubectl get netpol -A` **shows network policies for all namespace**
 - `kubectl explain ing.spec.rules.http` **show documentation of ingress for ing.spec.rules.http
 - 