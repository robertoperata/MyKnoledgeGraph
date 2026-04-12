---
tags: 
feature: 
type: 
author: 
source: 
---
# Pulumi Infrastructure Overview

## **Project Structure**

```
myuber-infra/
├── Pulumi.yaml                    # Project definition
├── index.ts                       # Main entry point
├── config/
│   ├── local.yaml                 # Local environment config
│   ├── aws-dev.yaml              # AWS dev config
│   └── aws-prod.yaml             # AWS prod config
├── components/
│   ├── k8s-cluster.ts            # Abstract K8s cluster
│   ├── postgres.ts               # Postgres component
│   ├── keycloak.ts               # Keycloak component
│   └── redis.ts                  # Redis component
└── environments/
    ├── local.ts                   # Local-specific resources
    └── aws.ts                     # AWS-specific resources
```

## **Single Script Approach**

**Main entry point (`index.ts`):**

- Read environment from `pulumi config get environment`
- Conditionally import local.ts or aws.ts
- Each environment file exports standardized component interfaces
- Components use Helm charts where possible

## **Environment Abstraction Pattern**

**Interface-based components:**

- Each component (postgres, keycloak, redis) exports same interface
- Implementation differs per environment (local vs AWS)
- Local: uses k3d + local storage
- AWS: uses EKS + RDS/ElastiCache managed services

## **Helm Integration Strategy**

**Prioritize Helm charts:**

- Keycloak: Official Bitnami Helm chart
- Redis: Bitnami Redis chart
- Postgres: Bitnami PostgreSQL chart (local) + AWS RDS (cloud)
- Use `k8s.helm.v3.Release` resources
- Override values per environment

## **Configuration Management**

**Stack-based configs:**

```bash
pulumi stack select local
pulumi stack select aws-dev
pulumi stack select aws-prod
```

**Environment-specific values in config files**

- Database instance sizes
- Storage classes
- Replica counts
- Resource limits

## **Deployment Commands**

**Single script deployment:**

```bash
# Local
./deploy.sh local

# AWS environments  
./deploy.sh aws-dev
./deploy.sh aws-prod
```

**Script handles:**

- Stack selection
- Config file loading
- Pulumi up execution
- Post-deployment validation

## **Teardown Strategy**

**Critical for AWS cost control:**

**Automated teardown script:**

```bash
./teardown.sh aws-dev --confirm
```

**Teardown approach:**

- Pulumi destroy with dependency ordering
- Explicit deletion of persistent volumes
- AWS-specific cleanup (load balancers, security groups)
- Cost validation queries before/after

**Safety mechanisms:**

- Confirmation prompts for production
- Dry-run mode
- Resource tagging for identification
- Cost estimation before teardown

## **Key Implementation Points**

**Local Environment:**

- k3d cluster creation
- Local storage classes
- NodePort services
- Minimal resource allocation

**AWS Environment:**

- EKS cluster with managed node groups
- AWS-native services where beneficial (RDS vs self-hosted)
- ALB ingress controller
- Production-grade resource allocation

**Cost Management:**

- All AWS resources tagged with environment/project
- Spot instances for dev environments
- Automatic shutdown schedules
- Resource cleanup verification

Want to dive deeper into any specific aspect?