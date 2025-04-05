Kubernetes Components: 

Control Plane Components
Manage the overall state of the cluster:

kube-apiserver: The core component server that exposes the Kubernetes HTTP API
etcd: Consistent and highly-available key value store for all API server data
kube-scheduler: Looks for Pods not yet bound to a node, and assigns each Pod to a suitable node.
kube-controller-manager: Runs controllers to implement Kubernetes API behavior.
cloud-controller-manager (optional): Integrates with underlying cloud provider(s).

Node Components
Run on every node, maintaining running pods and providing the Kubernetes runtime environment:

kubelet: Ensures that Pods are running, including their containers.
kube-proxy (optional): Maintains network rules on nodes to implement Services.
Container runtime: Software responsible for running containers. Read Container Runtimes to learn more.

Nodes: Kubernetes runs your workload by placing containers into Pods to run on Nodes. A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods.

A Kubernetes cluster consists of two types of nodes: master and worker nodes. 
The master node hosts the Kubernetes control plane and manages the cluster, including scheduling and scaling applications and maintaining the state of the cluster. 
The worker nodes are responsible for running the containers and executing the workloads.

image: master_wroker_node.PNG

What Is A Kubernetes Pod?
A Kubernetes pod is a collection of one or more application containers.
The pod is an additional level of abstraction that provides shared storage (volumes), IP address, communication between containers, and hosts other information about how to run application containers. 
So, containers do not run directly on virtual machines and pods are a way to turn containers on and off.

image node.PNG

What Is A Kubernetes Cluster?
Nodes usually work together in groups. A Kubernetes cluster contains a set of work machines (nodes). The cluster automatically distributes workload among its nodes, enabling seamless scaling.


Command line tool (kubectl)
Kubernetes provides a command line tool for communicating with a Kubernetes cluster's control plane, using the Kubernetes API.
Syntax: kubectl [command] [TYPE] [NAME] [flags]

where command, TYPE, NAME, and flags are:

command: Specifies the operation that you want to perform on one or more resources, for example create, get, describe, delete.
TYPE: Specifies the resource type. Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. 
NAME: Specifies the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed, for example kubectl get pods.
flags: Specifies optional flags. For example, you can use the -s or --server flags to specify the address and port of the Kubernetes API server.