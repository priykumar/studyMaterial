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
**Namespace Scope**
```
# 📦 Resources lives inside namespaces
kubectl api-resources --namespaced=true

# Resources global to entire cluster
kubectl api-resources --namespaced=false
```

### ⚠️ Notes
- Resources in same namespace can reference each other directly
- Cross-namespace communication requires FQDN: service.namespace.svc.cluster.local
- Deleting a namespace deletes everything inside it!

---
---

## 🟢 Pod
> The smallest deployable unit in Kubernetes

A Pod is a group of one or more containers with shared storage, network resources, and a specification for how to run them
- 📦 **Smallest K8S Unit** - You don't work directly with containers
- 🛡️ **Container Wrapper** - Abstracts away container runtime (Docker, etc.)
- 🎪 **Single Application** - One main app per pod (best practice)
- 🌐 **Shared Network** - All containers share the same IP and storage
- ⚡ **Ephemeral Nature** - Pods die easily and get replaced frequently

### 🎛️ Workload Resources that manage Pods
- 🚀 **Deployments** - Stateless applications
- 📊 **ReplicaSets** - Maintain pod replicas
- 🏢 **StatefulSets** - Stateful applications
- ⏰ **Jobs** - Run-to-completion tasks
- 🔄 **DaemonSets** - One pod per node

### 🌐 Pod Networking & Communication
- 🏠 **Within a Pod**
```
# Containers share localhost and storage volumes
curl localhost:8080  # Container A talking to Container B
```
- 🌍 **Between Pods**
```
# Each Pod gets unique IP address
Pod-A (IP: 10.244.1.5) → Pod-B (IP: 10.244.1.8)
```
* ⚠️ **The Ephemeral Problem**
```
Pod-A → Pod-B (10.244.1.8) ✅ Working
Pod-B crashes and restarts...
Pod-A → Pod-B (10.244.1.12) ❌ Broken! (New IP)

Solution: Services - Services provide stable endpoints for ephemeral pods!
```

### 🔄 Pod Lifecycle & States

| Phase          | Status   | What's Happening                                      |
|----------------|----------|-------------------------------------------------------|
| ⏳ Pending     | Starting | Accepted by cluster, waiting to run                   |
| 🟢 Running     | Active   | Bound to node, containers running                     |
| ✅ Succeeded   | Complete | All containers terminated successfully                |
| ❌ Failed      | Error    | Containers terminated with errors                     |

### 🔄 Restart Policies & Crash Handling
#### 🎛️ Restart Policies:
```
  restartPolicy: Always           # Default - always restart
  # restartPolicy: OnFailure      # Restart only on failure
  # restartPolicy: Never          # Never restart
```
#### 🔄 Crash Recovery Flow:
- 💥 Initial Crash → Immediate restart attempt
- 🔄 Repeated Crashes → Exponential backoff delay
- 😵 CrashLoopBackOff → Pod stuck in restart loop
- ⏰ Backoff Reset → After 10min success, reset delay

### 📦 Pod-related Kubernetes Commands
#### 🔍 View Pods:
```
kubectl get pods
kubectl get pods -o wide            # show more details (node, IP, etc.)
kubectl get pod <pod-name> -n <ns>  # view pod in specific namespace
```
#### 📖 Describe Pods:
```
kubectl describe pod <pod-name>
```
#### 📜 Logs
```
kubectl logs <pod-name>
kubectl logs -f <pod-name>                      # stream logs
kubectl logs <pod-name> -c <container-name>     # logs for a specific container
```
#### 🚪 Exec into a Pod
```
kubectl exec -it <pod-name> -- /bin/bash   # or sh
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash
```
#### 🛠️ Debugging
````
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod-name>       # check pod conditions and events
kubectl exec -it <pod-name> -- env    # check env vars inside pod
````
#### 📝 Create Pods
````
kubectl run nginx --image=nginx
kubectl run mypod --image=busybox --restart=Never --command -- sleep 3600
````
#### 🗑️ Delete Pods
````
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> -n <namespace>
kubectl delete pods --all             # delete all pods in current namespace
````
#### 🚦 Scale Pods (via Deployment)
````
kubectl scale deployment <deployment-name> --replicas=5
````

## ⚡ Static Pods
> Static Pods are a special type of Pod that are managed directly by the kubelet rather than the control plane. Static Pods are automatically created when the kubelet starts and are deleted when the kubelet shuts down. These Pods are not created or managed through the Kubernetes API server, but instead are defined by placing a Pod specification file on a node’s filesystem (/etc/kubernetes/manifests/)

---
---

## ⚖️ Services
>Stable networking for ephemeral pods - The solution to the IP address chaos!

A Service is an abstract way to expose applications running on a set of pods. It provides a single DNS name and load balances traffic across healthy pods   

### 🔗 Key Benefits:
- 📌 Permanent IP - Attaches stable IP to pods
- 🔄 Independent Lifecycle - Service survives pod deaths
- 🌐 Pod-to-Pod Communication - Pods talk through services
- ⚖️ Load Balancing - Distributes requests to least busy pods

**Problem → Solution**:
```
❌ Pod-A → Pod-B (10.244.1.8) → Pod-B dies → New Pod-B (10.244.1.12) → ❌ Broken!
✅ Pod-A → Service-B (stable DNS) → Any healthy Pod-B → ✅ Always works!
```

### 🔧 Service Types
#### 1. 🏠 ClusterIP
- Default service
- Visibility is internal-only that means used only for inter-pod communication
- It's not possible to use clusterIP service to reach a micro-service from the internet from outside the cluster
```
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP  # Default - can be omitted
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: backend-app
```

#### 2. 🚪 NodePort
- Visibility is internal and external to the cluster
- To use this, nodes must have public IP addresses
- Port must be in the range between 30000 and 32767 and if not provided then Kubernetes will assign it randomly
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  ports:
  - port: 80                # Internal cluster port
    targetPort: 8080        # Pod container port  
    nodePort: 30000         # External node port (30000-32767)
  selector:
    app: web-app
```
##### Traffic Journey:
- 👤 Client → http://node-ip:30000
- 🖥️ Node → forwards to Service:80
- ⚖️ Service → selects healthy Pod:8080
- 🟢 Pod → processes request
- 📤 Response travels back the same path
- _❌ No pods listening on 8080 → "Connection Refused"_
- _❌ No pods behind service → "Connection Timeout"_


#### 3. ☁️ LoadBalancer
- Visibility is external to the cluster
- K8S provision a logical LB and assign a public IP address to it so that external user access service through this LB IP
```
apiVersion: v1
kind: Service
metadata:
  name: production-web
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: web-app
```

| Type          | Visibility             | IP Access               |
|---------------|------------------------|-------------------------|
| 🏠 ClusterIP  | Internal only          | Cluster internal IP     |
| 🚪 NodePort   | Internal + External    | Node IP + Port          |
| ☁️ LoadBalancer | External             | Public cloud LB IP      |

### 📦 Service-related Kubernetes Commands
#### 🔍 View Services:
```
kubectl get svc
kubectl get svc -o wide                   # show cluster IP, external IP, ports
kubectl get svc <service-name> -n <ns>    # view service in a namespace
```
#### 📖 Describe Service:
```
kubectl describe svc <service-name>
```
#### 📝 Create Services:
```
kubectl expose pod nginx --port=80 --target-port=80
kubectl expose deployment myapp --type=NodePort --port=80
```
#### 🌐 Access Services:
```
kubectl get endpoints <service-name>    # see which Pods are behind the Service
kubectl port-forward svc/myservice 8080:80
```
#### 🗑️ Delete Services:
```
kubectl delete svc <service-name>
kubectl delete svc --all
```

### 🌐 Service Discovery & DNS
> When you create a service, Kubernetes automatically creates a DNS entry: ```<service-name>.<namespace-name>.svc.cluster.local```

#### 🏠 Same Namespace Access
```
# From any pod in the same namespace
curl -v http://my-svc:7780                    # DNS name
curl -v http://<my-svc-ClusterIP>:7780        # Direct IP
```
#### 🌍 Cross-Namespace Access
```
# From pod in different namespace to service in 'dummyns'
curl -v http://my-svc.dummyns.svc.cluster.local:7780  # FQDN
curl -v http://<dummyns-my-svc-ClusterIP>:7780        # Direct IP
```
#### 🖥️ External Cluster Access
```
# From outside the cluster - ClusterIP is NOT accessible!
# ❌ curl -v http://<my-svc-ClusterIP>:7780  # This won't work!

# ✅ Use NodePort or LoadBalancer services instead:
curl -v http://<node-ip>:30000               # NodePort
curl -v http://<loadbalancer-ip>:80          # LoadBalancer
```



