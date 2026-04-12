---
tags:
feature:
type:
author: "[[Sander van Vugt]]"
source:
---
# Certified Kubernetes Application Developer


## Module 4: Services and Networking
###  ***Kubernetes Networking***

Nodes are connected to physical external network infrastructure.
Kuberentes creates 2 software-defined networks:
- **cluster network**: is an <u>isolated</u> internal network that is used by the cluster for administrative purposes.
- **pod network**: internal software network to access the applications. when creating a pod it has an ip-address on the pod network. But these IP-addresses cannot be reached externally
**Services**: act like a load balancer are connected to the cluster network and are configured to expose pods (the endpoint). By default the service are not accessible from outside. 
Services can have different types that affect their behaviour:
- **ClusterIp**: internal type (default)
- **NodePort**: external type. That means that Kubernetes exposes a port (around 32000) in all Nodes

>[!question] How an external user connect to the appliction (pod) using NodePort?
>The client send a request through an external DNS that involves a Load Balancer that that balances the requests to the external nodes ports.
>The traffic will be forwarded to the service that will do his load balancing to the pods.
>The ClusterIp allows to connect Pods internally the pod network

normally to expose internal network is not used NodePort but instead **Ingress**, or **GatewayAPI** that are a revers HTTP proxy.

>[!question] How an external user connect to the appliction (pod) using Ingress or GatewayAPI?
The client send a request through an external DNS that connects to Ingress that forward to a service with ClusterIP


![[Kubernetes network overview|100%]]

 
 ### ***Services***
- A Service is an API resource that is used to expose a set of Pods.
   Kubernetes Services are all about networking
 - Services are applying Round-Robin load balancing to forward traffic to specific Pod
 - The set of Pods that is targeted by a Service is determined by a selector (which is a label)
 - The [[cloud-controller-manager]] will continuously scan for Pods that match the selector and include these in the Service
 - if Pods are added or removed they immediately show up in the Service
 - Services are <u>decopled</u> from the resources are referring to, they exist independently from the applications they provide access to
 - The Service needs to be created independently of the application, and after removing an application it also need s to be removed separately
 - **The only thing they do is watch for Pods that have a specific label set matching the selector that is specified in the service**
 - That means that ons Service can provide access to Pods in multiple Deployments, and while doing so Kubernetes will automatically load balance between these Pods
 - this strategy is used in canary Deployments
 
	 **Service Types**
 - **ClusterIP**: this default type exposes the service on an internal cluster IP address
 - **NodePort**: allocates a specific port on the node that forwards to the service IP address on the cluster network
 - **LoadBalancer**: provisions an <u>external load balancer</u> to handle incoming traffic to applications in public cloud. External load balancer means that it's needed to install something in addition to take advantage of it
 - **ExternalName**: works on DNS names; redirection is happening at DNS level which is useful in migration
 - **Headless**: a service used in cases where direct communication with Pods is required which is used in StatefulSet 

**Service Ports**
- while working with Services different ports are specified:
	- **targetPort**: the port on the application (container) that the Service addresses
	- **port**: the port on which the Service is accessible
	- **nodePort**: the port that is exposed externally while using the NodePort Service type

Service.yaml
the name is **nginxsvc** and is bind to the deployment (pod) with label **app=nginxsvc**
"Selector" is the key.
The IPs refers to the pods that has been scaled to 3
TargetPort is the port of the application
Port is the port of the service: \<unset> because there isn't a NodePort
```
kubectl describe svc nginxsvc

Name:              nginxsvc
Namespace:         default
Labels:            app=nginxsvc
Annotations:       <none>
Selector:          app=nginxsvc
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.215.61
IPs:               10.100.215.61
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.0.9:80,10.244.0.10:80,10.244.0.8:80
Session Affinity:  None
Internal Traffic Policy: Cluster
Events:

```

```yaml
kubectl get svc nginxsvc -o yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-08-25T18:42:30Z"
  labels:
    app: nginxsvc
  name: nginxsvc
  namespace: default
  resourceVersion: "1786"
  uid: d728aaf9-3118-4917-8ac7-5241df7e100a
spec:
  clusterIP: 10.100.215.61
  clusterIPs:
  - 10.100.215.61
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginxsvc
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

```

**Demo**
To verify to be able to connect to nginx we must:
- enter in the cluster because the pod is not visible from the outside due to the fact that the service is of type **ClusterIP**
- from the inside we can curl  the ip address (to discover the ip address `kubectl get svc`)
- from the outside is not reachable 
- Change from **ClusterIP** to **NodePort** `kubectl edit svc nginxsvc`
- get the ip of minikube node `minikube ip`
- get the port exposed by NodePort `kubectl get svc`
- now is possible connect to the Pod: `curl 192.168.49.10:30635`

### Services and DNS
- Exposed Services automatically register with the Kubernetes internal [[CoreDNS (or kube-dns)|coredns]] server
- The standard DNS name is composed as `servicename.namespace.svc.clustername`
- as a result, Pods within the same Namespace can access service-name by using its short name
- to access service-names in other Namespaces the fully qualified domain name must be used
**Demo**
- describe the service of [[CoreDNS (or kube-dns)|kube-dns]] with `kubectl describe -n kube-system kube-dns`
- it shows the IP address and the port 53 it's listening 
- create a new namespace: `kubectl create ns othernamespace`
- create a pod `kubectl run mypod -n othernamespace --image=nginx 
- create a service to port-forward mypod `kubectl expose -n othernamespace pod mypod --port=80`
- create a pod in default namespace to connect to mypod in other namespace: `kubectl run testpod --image=busybox -- sleep inifinty`
- use testpod to print what inside its `/etc/resolv.conf` : `kubectk exec -it testpod -- cat /etc/resolv.conf`

```bash
nameserver 10.96.0.10 
search default.svc.cluster.local svc.cluster.local cluster.local options ndots:5
```

`nameserver 10.96.0.10` is the IP address of [[CoreDNS (or kube-dns)|kube-dns]] 
`search deafult.svc.cluster.local` is the complete domain for those looking from other namespace

to connect to `mypod` in `othernamespace` from `testpod` in `dafault` namespace: `kubectl exec -it testpod -- wget --spider --timeout=1 mypod.othernamespace.svc.cluster.local`

### NetworkPolicy

- By default there are no restriction to network traffic in K8s
- Pods can always communicate even if they're in other Namespaces
- To limit this NetworkPolices can be used
- NetworkPolicies need to be supported by the network plugin though
	- the Wave plugin does not support NetworkPolicy
	- Calico is a common plugin that does support NetworkPolicy
- if in a policy there is no match traffic will be denied
- in fo NetworkPolicy is used all traffic is allowed

**NetworkPolicy identifiers**
- In NetworkPolicy three different identifiers can be used:
	- **podSelector**: specified a label to match Pods
	- **namespaceSelector** used to grant access to specific namespaces
	- **ipBlock**: marks a range of IP addresses that is allowed. notice that traffic to and from the node where a Pod is running is always allowed
- When defining a Pod or Namespace-based NetworkPolicy a selector label is used to specify what traffic is allowed to and from the Pods that match the selector
- NetworkPolicies no not conflict, they are additive

### Advanced Networking: Gateway API and Istio
- Gateway API provides routing and traffic management policies
- it is an advanced layer that uses custom resources for managing incoming (ingress) and outgoing (egress) traffic
- gateway API adds features that are note addressed by Ingress, including:
	- support for TCP, UDP and gRPC
	- traffic splitting and mirroring
	- Websocket protocols
- Future developments seem to further integrate Gateway API functionality with Ingress
- You'll learn more about Gateway APUI in the next lesson

**What is Istio**
- Istio is a service mesh and makes managing complex relations between applications in a microservice easier.
- as  such ti provides rules for managing traffic in microservices
- it includes features for traffic management, security, and observability in the service mesh
- its focus is on service-to-service communication
- Istio may be used in addition to core Kubernetes networking and is optional
