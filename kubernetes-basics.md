# Kubernetes and Docker
At its core, K8S is a container orchestrator. One key facet is workload placement.  
K8S also provides an infrastructure abstraction.  

Benefits of K8S:
- Speed of development
- Ability to absorb change quickly
- Ability to recover quickly
- Hide infrastructure complexity in the cluster

Principles of K8S:
- Desired state/Declarative configuration
- Controllers/Control loops
- API/The API Server

## Key Kubernetes API Objects
Building blocks for deployments.

### Pods
A single or a collection of containers. Essentially our container-based applications.  
This is the most basic unit of work, i.e. the unit of scheduling.
Pods are never redeployed, they are ephemeral. No state is maintained between pod deployments.  
Pods are atomic. They're either there or they're not.

Pod state and health is tracked by Kubernetes by using "Probes". A probe can perform health checks against an url to see how e.g. a web app responds. 

### Controllers
A controller keeps our system in the desired state.  

Commonly used to create and manage pods for your. Controllers respond to Pod state and health.  
`ReplicaSet` allows us to define a number of replicas of a specific pod.

ReplicaSets are created by what's defined in a deployment. Deployment controller manages the transition between ReplicaSets, this happens e.g. when moving between two versions of an app.

### Services
Persistent access point to the applications we deploy. E.g. as pods are torn down and redeployed, services ensure persistent access points, so we don't have to change IPs, etc.  
Services provide networking abstraction for Pod access.  

Provides functionality to scale applications, i.e. adding/removing Pods, as well as load balancing.

### Storage
Persistent storage for applications.  
Define persistent volume claim in the Pod definition, e.g. "this pod need 10GB of storage".


## Kubernetes architecture

### Control plane node
Major control function of a cluster.
- Has an API server: Main access point for cluster administration. This is the communication hub for the k8s cluster. Runs on port 6443.
- Cluster store: `etcd` is responsible for storing the cluster objects' state. Runs on port 2379-2380 and is used by the API server.
- Scheduler: tells which nodes to start pods on. Runs on port 10251.
- Controller manager: keep things in desired state. Runs on port 10252.
- Kubelet: runs on port 10250.

`kubectl` is an CLI program designed to interact with the API Server.

### Node
aka. Worker node. Either virtual or physical machines.
Application pods run here. A node is responsible for starting up pods.
Kubelet on worker node runs on port 10250.
NodePort = ports 30000-32767.


## Deployments
Define a yaml [deployment file](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to specify what goes into kubernetes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
    tier: backend

# The important part comes now
spec:
  selector:
    matchLabels: # selector should match label inside the template label
      tier: backend
  template:
    metadata:
      labels:
        tier: backend
    spec:
      containers:
      - name: my-app
        image: nicm/my-app-image
        imagePullPolicy: IfNotPresent # Include this on local development
        ports:
          - containerPort: 8080
```

The deployment.yaml is sent to the API server using `kubectl`.  
To create the deployment, run the following command:  
`kubectl create -f path.to.deployment.yaml --save-config` or use `kubectl apply` instead of `create`.
`apply` can both create and modify.

Note: at this point, there's no access to the created pod. A `service` is required for this.

### Deployment options
1. Delete all existing Pods and replace with new Pods. This leads to short down-time.
2. Start new Pods and then delete old Pods. Need to be able to run two versions simultaneously.
3. Replace existing Pods one by one without impacting traffic to Pods (aka. Rolling update, default).
4. Blue-Green: Run both old and new version simultaneously.
5. Canary: Only small subset of traffic is routed to the new version.
6. Rollback: 

The `.spec.strategy` object is used to specify how old Pods are replaced by new ones.


## Services
Services can be defined in their 

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  labels:
    tier: backend
spec:
  type: LoadBalancer # This type allows traffic outside of the k8s cluster
  selector:
    tier: backend # select the pods with this label
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

