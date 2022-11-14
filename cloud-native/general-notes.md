# Cloud Native

## Scalability
Cloud native applications should be:
- Scalable
- Stateless
- Observable
- Secure
- Platform agnostic: portable
- Addressable: internally or publicly

## Containerization
Use containers to allow autoscaling and portability. Containers can run on any container-enabled platform.
This also helps the "works on my machine" issue.

Azure offers "Azure Container Apps" to run applications without the overhead of managing the underlying platform.

https://learn.microsoft.com/en-us/azure/container-apps/overview

## Service discovery
Service discovery, or, addressability, is the process of resolving a microservice's location.  
For a microservice arcitechture to work, the services needs to be addressable.

Microservices often need to call each other within the same network, making sure the requests never leave the environment. This can be done with using Fully Qualified Domain Names (FQDN) or Dapr ([example here](https://github.com/Azure-Samples/container-apps-connect-multiple-apps))


- https://learn.microsoft.com/en-us/azure/container-apps/connect-apps?tabs=bash
- https://microservices.io/patterns/service-registry.html
- https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/microservices-addressability-service-registry

## Security
### Secrets
Applications often need configuration values that are considered secret and are not to be committed to the git history. Also, secrets are very likely to change between environments, so, they always be external to the application.

https://learn.microsoft.com/en-us/azure/container-apps/manage-secrets?tabs=arm-template

## Observability


## Performance
### TLS termination
TLS has some overhead as traffic needs to be encrypted and decrypted.  
You can use a proxty that offers TLS termination to avoid the overhead at application level.

https://learn.microsoft.com/en-us/azure/container-apps/ingress?tabs=bash

## Kubernetes

Azure Kubernetes Service (AKS)
- https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks-microservices/aks-microservices-advanced

## Dapr
Short for "Distributed Application Runtime", a framework for building protable and reliable microservices.
Dapr is not language-specific and uses sidecars and can run in two modes: 1) self-hosted or kubernetes.

https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview?tabs=bicep1%2Cyaml

## Continous Deployment
### GitOps
https://www.gitops.tech/

## Other links
- https://github.com/Azure/reddog-code