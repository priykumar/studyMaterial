# Kubernetes
  - an open-source system  
  - helps to deploy, manage, and scale applications – especially those running in containers like Docker  
  - manages containers
      | Action                           | Concept          |
      |----------------------------------|------------------|
      | start and stop them              | Deployment       |
      | restart them when they crash     | Self healing     |
      | scale them when traffic increases| Auto scaling     |
      | balance traffic across them      | Load balancing   |
      | update them without downtime     | Rolling updates  |
   
# Architecture
Consist of 2 types of nodes
  - Master node
  - Worker node

### Master node 
  - Brain of the cluster
  - Performs all the managing tasks like managing worker nodes and pods in the K8S cluster
  - Following four processes run on all the Master nodes to control cluster state and the worker nodes
    * API server:
        * Cluster gateway that accepts all the request coming to the cluster
        * Act as a gatekeeper, which authorise and authenticate your request
        * Exposes K8S API
        * Designed to scale horizontally — that is, it scales by deploying more instances
    * Scheduler:
        * Accepts request from API server
        * Decides on which particular node the next pod or other component will be scheduled
        * After deciding the node, it sends the request to the kubelet of that particular node and kubelet is the one which deploys that new resources on the node
    * Controller manager:
        * Monitors the cluster state continuously, and trues to maintain current state same as manifested state.
        * For example, if a pod dies, then it tries to create a replica of that pod as soon as possible. It sends request to scheduler to create the new pod and then scheduler checks on which node new pod should be deployed and send the request to kubelet of that pod
    * Etcd:
        * Consistent and highly-available key value store, that stoes cluster state
        * Any change in the cluster is noted down by the etcd and stored as key value pair
        * Scheduler and control manager gets its data from etcd and based on its data it act upon it. Data stored here are very useful because the dealer and control manager act upon it, so it must be stored safely. So it’s recommended to have multiple master so that if one master goes down, we dont lose the data
     
### Worker node 
  - Actually does some work
  - Following three processes run on all the worker nodes
    * Container runtime - Docker
    * Kubelet:
      * Informs the API server on master node that the node exists and is ready
      * Schedules the pod and container running inside the pod
      * Responsible for creating pod and container and assigning the resources to these from the node, like CPU, RAM and storage resources
      * Checks the health of pods and containers via liveness/readiness probes
      * Keeps track of resource usage on the node
      * Command to check if kubelet in running: ```ps aux | grep kubelet```
      * Kubelet logs: ```journalctl -u kubelet```
      * Config path: ```/etc/kubernetes/kubelet.conf```   
    * Kube-proxy:
      * Traffic manager on each node that helps route requests to the correct pod across the cluster
      * Responsible for forwarding the request from services to pods. If pod A needs to send the request to pod B, then kube proxy will direct request to pod B on the same node instead of sending it to the other replica to avoid the network overhead.
      * Let’s say my-service selects 3 Pods: pod-a, pod-b, and pod-c   
        * A request comes to my-service:80.
        * kube-proxy sees it and uses iptables to forward it to one of the Pods (say pod-b).
        * If pod-b dies, it updates the routing rules and stops sending traffic to it.
       
### Communication between master and worker nodes
  - **Worker → master**
    * All communication between nodes/pods and the control plane goes through the API server. Control plan doesn't expose any other component  
    * The API server uses HTTPS (usually port 443) to communicate, make sure connections are secure  
    * Nodes need valid credentials (like a client certificate) to connect securely  
    * Pods can also connect to the API server using a service account that automatically gives them the required credentials (like a token and certificate)  
    * kubelet -> API Server : Register the node, report Pod status  
    * Kube-proxy -> API Server: Watch services and Endpoints
   
  - **Master → worker**
    * API Server -> kubelet
      * Fetching pod logs
      * Attaching Running pods to node
      * Providing kubelet's port forwarding functionality
    * API server -> Nodes, pods, services
      * API server connections to nodes, pods, or services default to plain HTTP (not encrypted or authenticated).
      * These connections can be switched to HTTPS by prefixing https: to the URL.
      * The HTTPS connection will be encrypted, but:
          * It does not validate the certificate of the HTTPS endpoint.
          * It does not provide client credentials for authentication.  
  
