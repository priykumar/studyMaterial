# ⚓ Kubernetes
> 🚀 An open-source container orchestration platform that helps deploy, manage, and scale applications – especially those running in containers like Docker


## 🎯 Why Kubernetes?
    
  | What Your App Needs        | Kubernetes Magic                        | Technical Term  |
  |----------------------------|-----------------------------------------|-----------------|
  | Start and stop containers  | Spins up/down apps instantly            | Deployment      |
  | Handle crashes gracefully  | Auto-restarts failed containers         | Self Healing    |
  | Scale during traffic spikes| Adds more instances automatically       | Auto Scaling    |
  | Balance traffic            | Routes traffic intelligently            | Load Balancing  |
  | Update without downtime    | Gradual rollouts with zero interruption | Rolling Updates |
   
## 🏗️ Architecture

<img width="1400" height="817" alt="-000" src="https://github.com/user-attachments/assets/b24c9f50-1c0d-4308-8f3c-68484696a0a1" />

K8S cluster consists of 2 types of nodes:
  - Master node
  - Worker node

### 👑 Master Node 
> 🧠 The brain of the cluster - Performs all managing tasks for worker nodes and pods

Four core processes control the entire cluster:
  - 🚪 **API server** - _The Gatekeeper_
    * **Primary function**: Cluster gateway that accepts all requests to the cluster
    * Act as a gatekeeper, which authorise and authenticate requests
    * Designed to scale horizontally by deploying multiple instances
    * Exposes kubernetes APIs
    * Communication: HTTPS on port 443
     
- 📋 **Scheduler** - _The Advisor_
    * **Primary function**: Decides which node will run new pods or components
    * Receives requests from API server
    * Evaluates node resources and constraints and decides target node
    * Sends deployment requests to target worker node's kubelet
     
- 🎛️ **Controller manager** - _The Inspector_
    * **Primary function**: Monitors cluster state continuously
    * Maintains current state matching the desired (manifested) state
    * Multiple controllers handle different resource types (ReplicaSet, Deployment, etc.)
    * Example: When a pod dies, immediately requests scheduler to create a replacement
      
- 📚 **Etcd** - _The Treasurer_
    * **Primary function**: Consistent, highly-available key-value store for cluster state
    * Stores all cluster configuration and state changes
    * Scheduler and Controller Manager rely on etcd data
    * Multiple master nodes recommended to prevent data loss
     
### 🏭 Worker Node 
> 💪 Where the real work happens - Runs the actual application workloads

Three essential processes on every worker node
  - 🐳 **Container runtime** - _The Factory Floor_
    * Manages container lifecycle (pull images, start/stop containers)
    * Default: Docker (also supports containerd, CRI-O)
      
  - 🤖 **Kubelet** - _The Floor manager_
    * **Primary function**: Node agent that communicates with API server
    * Responsibilities:
      * Registers node in kubernetes cluster and reports its readiness
      * Schedules pods and manages containers within pods
      * Allocates node resources (CPU, RAM, storage) to pods
      * Performs pod health checks via liveness/readiness probes
      * Tracks resource usage and reports to API server
     
    * Check if kubelet is running → ```ps aux | grep kubelet```
    * View kubelet logs → ```journalctl -u kubelet```
    * Kubelet configuration → ```/etc/kubernetes/kubelet.conf```
           
  - 🚦 **Kube-proxy** - _The Traffic Director_
    * **Primary function**: Network traffic manager for each node
    * Routes service requests to appropriate pods
    * **Optimization**: If pod A needs to send the request to pod B, then kube proxy will direct request to pod B on the same node instead of sending it to the other replica to avoid the network overhead
    * Load balancing example:
      * Service _my-service_ targets pods: pod-a, pod-b, pod-c
      * Incoming request to _my-service_:80
      * kube-proxy uses iptables to forward to available pod (e.g., pod-b)
      * If pod-b dies, it updates the routing rules and stops sending traffic to it
       
## 📡 Communication Between Master & Worker Nodes
  - 🔄 **Worker → Master**
    * All communication goes through the API server, as control plane doesn't expose any other component  
    * Nodes use client certificates for authentication
    * Pods can also connect to API server using _service account_ that automatically gives them the required credentials (like a token and certificate)
    * Communication flows:
      * kubelet → API Server: Node registration, pod status reports
      * kube-proxy → API Server: Service and endpoint monitoring
   
  - 🔄 **Master → Worker**
    * Communication flows:
      * API Server → kubelet
        * Fetching pod logs
        * Attaching Running pods to node
        * Port forwarding functionality
      
      * API server → Nodes, pods, services
        * **Default**: Plain HTTP (unencrypted, unauthenticated)
        * HTTPS option: Available by prefixing https: to URLs
        * HTTPS connections are encrypted but don't validate certificates or provide client authentication 

## 🧱 K8S Objects
>The building blocks of your K8S cluster - Everything in Kubernetes is an object!

Common K8S Objects:
* 🟢 **Pods** - Smallest deployable units
* ⚖️ **Services** - Network access to pods
* 🚀 **Deployments** - Manage pod replicas
* 🔄 **ReplicaSets** - Ensure desired pod count
* 💾 **PersistentVolumes** - Storage resources
* ⚙️ **ConfigMaps** - Configuration data
* 🔐 **Secrets** - Sensitive information

### 📝 Creating K8S Objects
Multiple approaches available:
* 📄 Manifest files (Recommended)
* 📦 Helm charts
* ⌨️ kubectl commands
* 🏗️ Terraform
* 🔧 Other IaC tools
        
> Manifest file is a blueprint that tells Kubernetes what you want to create, configure, or modify. It is a YAML or JSON file that describes the desired state of an object

```
# 🔍 Validate manifest syntax
kubectl --validate -f <manifest-file>

# 🚀 Apply manifest to create/update objects
kubectl apply -f <file-name>

# 📁 Apply entire directory
kubectl apply -f ./manifests/

# 🗑️ Delete objects from manifest
kubectl delete -f <file-name>
```

### 📋 Manifest Structure     
A manifest should have atleast these fields - apiVersion, kind, metadata
```
apiVersion: v1          # API version for the object
kind: Pod               # Type of K8S object
metadata:               # Object identification
  name: my-awesome-pod
  labels:
    app: web-server
```

## 🌐 K8S API
> The heart of Kubernetes - Everything flows through the API!

Every operation, communication, and state change happens through this unified interface. There are sevral ways to interact with K8S APIs - kubectl, client-library tools, HTTP curl, Helm

```👤 You → 🛠️ kubectl/CLI → 🚪 API Server → 📚 etcd + ⚙️ controllers/workers```

---

## 📂 Namespace
> Virtual folders for your cluster - Keep your resources organized and isolated!

A namespace in Kubernetes is like a folder or project space that helps you organize and separate your resources (Pods, Services, Deployments, etc.)   
 
### 🏷️ **Pre-defined namespaces:**
* default
* kube-system
* kube-public
* kube-node-lease

### 🛠️ Namespace Commands
**Creating & Managing Namespaces:**
```
# 🆕 Create a new namespace
kubectl create namespace my-team

# 📋 List all namespaces
kubectl get namespaces

# 🔍 Get namespace details
kubectl describe namespace my-team
```
**Working with Specific Namespaces:**
```
# 👀 View resources in specific namespace
kubectl get pods -n my-team
kubectl get all -n kube-system

# 🎯 Set default namespace for current context
kubectl config set-context --current --namespace=my-team

# 🗑️ Delete namespace (⚠️ Deletes ALL resources inside!)
kubectl delete namespace my-team
```


Not all objects are in a namespace, like nodes, PVs. To see which K8S resources are and aint in namespace
* In a namespace: kubectl api-resources --namespaced=true
* Not in a namespace: kubectl api-resources --namespaced=false
