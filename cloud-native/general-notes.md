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

**Building blocks**:  
- State management
- Service to service invocation
- Pub-sub messaging
- Accessing secrets
- Bindings
- Observability

The sidecar approach is beneficial because it makes the sidecars talk to one another rather than the actual services.  
This means, we can get autoamtic retries, mTLS encryption, access control policies, tracing and metrics out of the box.

**Running on Kubernetes**  
Often used for production and requires you to install Dapr on the cluster.  
Requires microservices to be annotated in a special way to allow the Dapr Runtime to automatically create the sidecars.

**Sidecars**  
Each sidecar needs to bind to separate ports and can be interacted with through HTTP requests.  
A sidecar URL might look like `http://localhost:3500/v1.0/state/statestore` to access the default redis store.

### Getting started with Dapr
1. Install the Dapr CLI, e.g. `brew install dapr/tap/dapr-cli`
2. Install the Dapr Runtime by running `dapr init`. Installing it locally requires docker to be installed.

Installing the runtime locally creates a dapr deamon (daprd) and adds it to your path.

You can add a script like this to your application.
```bash
dapr run \
  --app-id frontend \
  --app-port 5266 \
  --dapr-http-port 3500 \
  dotnet run
```

https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview?tabs=bicep1%2Cyaml

## Continous Deployment
### GitOps
https://www.gitops.tech/

## Other links
- https://github.com/Azure/reddog-code