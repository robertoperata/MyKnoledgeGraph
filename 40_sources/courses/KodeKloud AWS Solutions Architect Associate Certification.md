---
tags:
  - aws
  - solutions-architect
  - cloud
  - certification
feature: 
type: course
author: KodeKloud
source: 
---

## VPC
VPCs allow us to isolate resources from other resources in the cloud.
VPCs give full control of networking in the cloud .
you as the customer get to define all of the **subnetting**. You get to specify what IP addresses you want to use in the cloud. 
You get to configure the **routing table** so you get to dictate where and what path packets take as it traverses through your AWS account. 
You get access to **firewalls**, so things like [[Security Groups vs NACLs|security groups and NACLs]], which ultimately determine what type of traffic you allow in and out of your environments as well as to your individual resources. 
You can customise traffic that goes in and out of your VPCs by utilising things called **gateways**. 
![[_- visual selection.svg]]

VPC are specific to a single region, VPC cannot span more and one per region. 
VPC acts as a network boundary. When you create resources in a VPC, by default they are not able to communicate with other resources in another VPC. ***They are automatically isolated by default***.
And if you want to allow either the **internet** to talk to resources in your virtual private cloud, or if you want to be able to allow communication between two **different VPCs**, you have to specifically allow them. 

Every VPC has a range of IP addresses assigned to it. This is called a CIDR block. 
A CIDR block defines the IP addresses that resources in the VPC can use. 
 a CIDR block can be of size anywhere between /16 to /28. 
 - you are allowed to optionally enable a secondary IPv4 for block if you like (optional)
 -  you can enable IPv6 CIDR block for your VPC, which gives you a /56 CIDR block (optional)
 -  and you can have up to five /16 CIDR block but the limit is adjustable (optional)

there are two kinds of VPC:

1. The default VPC
2. The custom VPC

The default VPC is created by AWS  in every region  that you have access to.
 this VPC has a default configuration that allows internet connectivity for your resources

 with custom VPC you are entitled to create a brand new VPC and you have to specify all the configuration, specifying the  CIDR block and different configuration settings

let's have a look at the default VPC created by Amazon 
- one default VPC per region
- /16 IPv4 block 172.31.0.0/16 -> 65.536 addresses
- for for every availability zone a default subnet is configured with /20
-  an internet gateway is attached to the VPC 
- A default route that's going to point all traffic to the internet gateway.(0.0.0.0/0).  this provides internet access for the resources within your default VPC
-  a default [[Security Groups vs NACLs|security group]] assigned to your resources, Allow outbound traffic.
- A default [[Security Groups vs NACLs|network access control (NACL) list]]  to secure your subnet and your VPC overall  open both inbound and outbound directions  

By default creates a subnet for each AZ in a Region.
Route table and Internet gateway are also created

Looking to the subnet, because there's a Internet Gateway assigned to this VPC **Auto-assign public IPv4 address: Yes** that's means that an EC2 instance in this subnet is public reachable.

## Subnets
subnets are groups of IP addresses within your VPC, that  you can use to deploy resources into.
subnets reside within a single AZ
subnets can be public or private using Internet Gateways and [[NAT Gateway|Nat Gateways]]
subnet must be inside the CIDR VPC range
a subnet block size must be within /16 and /28
the first 4 IP addresses of a subnet are reserved and cannot be used:
192.168.x.0 -> network address
192.168.x.1 -> AWS - VPC router
192.168.x.2 -> AWS - DNS
192.168.x.3 -> AWS - future use
192.168.x.255 -> broadcast address

subnet cannot overlap with other subnet in the same VPC
a subnet allows for optional IPv6 CIDR
a subnet can be configured to be IPv6 only - no IPv4 addresses
subnet can comunicate with other subnet in the VPC
auto-assign Public IPv4/IPv6 address in addition to the private address

### Routing
Every VPC has a VPC Router and this router has an interface in any subnet of the VPC and can be reachable from the IP Address + 1 of each subnet
so if our subnet is 192.168.1.0/24 the router's interface in the subnet would be 192.168.1.1
The purpose of the router is to route traffic between subnets as well as in and out of the VPC.
AWS allows to control the router by configuring the route table which determines where network traffic will get sent. So route table is a set of rules that the router uses to forward the network traffic, and each rule in the route table is referred to as a route.
In case two routes overlap AWS chose the route with a larger prefix length.
once the route is selected for the packet the VPC router will forward the network packet to the target field of that specific route.
Every subnet is associated with one route table by default but multiple subnet can be associated with a single route table.
 multiple subnets can be associated with one routing table but a subnet can only be associated with one route table in total 

### Internet Gateway
By default subnets are private, that means devices inside the subnet can't talk to internet and vice-versa.
To make subnet public we need to use **internet gateway**
Internet gateways are attached to VPC and are region resilient, which means they cover all Availability Zones within a region.
if a VPC doesn't have an Internet Gateway attached to it, than all subnets in that VPC are going to be considered private.
A VPC can only have 1 internet gateway attached and an internet gateway can only be attached to one VPC.
What we have to do to make subnet public?
- create Internet Gateway (IGW)
- attached to VPC
- create a custom route table
- configure default route pointing to the internet gateway
- associate the subnet to that route table
this give all resources in that subnet access to the internet.
when you deploy resources onto a public subnet by default it's only going to get a private IP. So you must enable to give a public IP.
The resource is aware only of his private IP, AWS point the public IP to the private IP of the resource. The public IP is associated to the private IP.

## Nat Gateway
[[NAT Gateway]] allows subnets to connect to the internet but connections must be initiated from within the VPC. Example a server that have to update packages.
NAT Gateway are deployed onto public subnet so that they have a public ip and internet access (with the route table pointing to the internet gateway and deploy the nat gateway on the public subnet).
uses Elastic IPs
Route table for private subnet should point to NAT Gateway 
billed per hour + per GB of data
the nat gateway is deployed on the subnet and if the subnet goes down you lose the nat gateway
Nat Gateways are AZ-resiliant service, need ! NAT Gateway in each AZ
Managed by AWS, supports 5 Gbps of bandwidth and automatically scales up to 100Gbps

![[AWS VPC Network Overview.excalidraw|10000]]

## Setup [[NAT Gateway]] — Ordine operativo

Per permettere a una risorsa in una subnet privata di accedere a internet tramite NAT Gateway, i passi da seguire in ordine sono:

### 1. Creare la VPC
- Definire il CIDR block (es. `10.0.0.0/16`)

### 2. Creare le subnet
- **Subnet pubblica** (es. `10.0.1.0/24`) — dove vivrà il NAT Gateway
- **Subnet privata** (es. `10.0.2.0/24`) — dove vive la risorsa che vuole accedere a internet

### 3. Creare e attaccare un Internet Gateway (IGW)
- Creare l'IGW
- Attaccarlo alla VPC (1 IGW per VPC)

### 4. Creare la route table per la subnet pubblica
- Aggiungere la route di default: `0.0.0.0/0 → Internet Gateway`
- Associare questa route table alla **subnet pubblica**

### 5. Allocare un Elastic IP
- Il NAT Gateway richiede un IP pubblico statico (Elastic IP)

### 6. Creare il NAT Gateway
- Deployarlo nella **subnet pubblica** (non in quella privata)
- Associargli l'Elastic IP

### 7. Creare la route table per la subnet privata
- Aggiungere la route di default: `0.0.0.0/0 → NAT Gateway`
- Associare questa route table alla **subnet privata**

### 8. Deployare la risorsa nella subnet privata
- La risorsa riceve solo un IP privato
- Il traffico outbound percorre: `risorsa → router VPC → NAT Gateway → IGW → internet`

> **Punto critico:** il NAT Gateway va nella subnet **pubblica**, non in quella privata. La subnet privata lo raggiunge tramite la route table. Se la subnet pubblica cade, perdi anche il NAT Gateway.

## DNS 
Device private IPs will automatically be assigned a DNS entry

AWS DNS server can be accessed on the second IP of the VCP CIDR block as well - 169.254.169.253

enableDnsHostNames: Determines whether the VPC supports assigning public DNS hostnames to instances with public IP address

enableDNSSupport : Determines whether the VPC supports DNS resolution through the Amazon provided DNS server

## Elastic IP
public ip are not static and if a EC2 instance goes down then it will get another IP
elastic IP are static IPv4 address that do not change
to use an ElasticIP you first allocate one to your account and then associate it with your instance or a network interface
Elastic IP are region specific

## Security Group and NACLs
Firewalls monitor traffic and only allow traffic permitted by a set of predefined rules
Firewalls rules are broken down into inbound and outbound rules
Stateless firewalls need to be configured both inbound and outbound
Stateful firewalls are intelligent enough to understand which request and response are part of the same connection. if a request is permitted the response is automatically permitted as well in a stateful firewall.
### Network Access Control List (Firewall of AWS)
NACLs filter traffic entering and leaving a subnet.
NACLs do not filter traffic within a subnet.
NACLs are stateless firewall so rules must be set for both inbound and outbound traffic
Allow or deny traffic
Every subnet in a VPC must be associated with a NACL.
You can associate multiple subnet to a single NACL but a subnet can only be associated to a NACL
### Security Groups
Security Groups act as firewalls for individual resources
Security Group are stateful so only request needs to be allowed
Only allow traffic
multiple security group are merged into one

## Load Balancer
- classic load balancer
	- able to handle only one SSL certificates
- application load balancer
	- support HTTP/HTTPS/WebSockets
	- function at the application layer (level 7)
	- forward requests based off of:
		- URL path conditions
		- Host domain
		- HTTP field - header, method, query, IP
		- support HTTP redirects and custom HTTP response
	- perform application-specific health checks
- network load balancer
	- laod balance traffic based on TCP/UDP (layer 4)
	- meant for applications that don't use HTTP/HTTPS
	- faster than Application load balancers
	- health checks are only basic ICMP/TCP connection
	- NLB forwards TCP connection to instances
![[Drawing 2026-04-25 11.23.08.excalidraw|1200px]]
### Cross-zone load Balancing
allow load-balancer nodes to balance through different AZ

## VPN Architecture in AWS

**VPN Gateway** terminates VPN on the AWS side.
**Customer Gateway** terminates VPN on the Customer side

## Direct Connect
Directly links on-premises with AWS without internet routing like VPN
Offers greater throughput and a more secure, stable connection than VPN over the internet
cross connect is a connection between a port on AWS router and customer router
pricing:
- port hours
- outbound data transfer

## VPC Peering
Network connection between two VPCs that routes traffic between them
VPC Peering connects VPCs in the same/different regions and AWS accounts
No charge for creating a VPC Peering, data transfer that crosses AZ is chargeable
 

## Transit Gateway
- simplify networking between VPCs and On-Premise environments
- allow for transitive routing
- must specify one subnet for each AZ to be used by the transit gateway to route traffic
- can peer with other Transit Gateways in different regions or AWS accounts
## Privatelink
- allows resources in our VPC to connect to services as if they are were in the same VPC
- used to connect to public AWS services (S3, CloudWatch) or to other VPCs in AWS
- VPC endpoints facilitate communication between  VPC instances and services
## CloudFront
- delivers content via a worldwide network of data centers called edge location
- helps get content close to customers
- origin is the content source for CloudFront edge locations
- Distribution is a unit of configuration for Cloudfront
- CloudFront distributions get a default domain name http://xyz.cloudfront.net
- TTL value decided content validity before an edge location requests the origin
- Default TTL is 24 hours
- Cache invalidation - invalidates objects at edge location before their TTL expires

## Argomenti mancanti

- [[VPC Peering]]
- [[VPC Endpoints]] (Gateway e Interface)
- [[AWS PrivateLink]]
- [[Transit Gateway]]
- [[Site-to-Site VPN|VPN Site-to-Site]] e Client VPN
- [[AWS Direct Connect]]
- [[VPC Flow Logs]]
