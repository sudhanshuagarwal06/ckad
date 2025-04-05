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


What Is A Kubernetes Pod?
A Kubernetes pod is a collection of one or more application containers.
The pod is an additional level of abstraction that provides shared storage (volumes), IP address, communication between containers, and hosts other information about how to run application containers. 
So, containers do not run directly on virtual machines and pods are a way to turn containers on and off.

image node.PNG

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80


Replication Controller:  A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available

apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80


ReplicaSet: A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

to scale the replicaset: kubeclt scale replicaset --replicas=<desire_number> -f replicaset_name

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5


+--------------------------------------------------+-----------------------------------------------------+
|                   Replica Set                    |               Replication Controller                |
+--------------------------------------------------+-----------------------------------------------------+
| Replica Set supports the new set-based selector. | Replication Controller only supports equality-based |
| This gives more flexibility. for eg:             | selector. for eg:                                   |
|          environment in (production, qa)         |             environment = production                |
|  This selects all resources with key equal to    | This selects all resources with key equal to        |
|  environment and value equal to production or qa | environment and value equal to production           |
+--------------------------------------------------+-----------------------------------------------------+

Labels and Selectors
Labels are key/value pairs that are attached to objects such as Pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key/value labels defined. Each Key must be unique for a given object.

"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}


Deployment: A Deployment manages a set of Pods to run an application workload, usually one that doesn't maintain state.
Use Case: The following are typical use cases for Deployments:
Create a Deployment to rollout a ReplicaSet. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
Declare the new state of the Pods by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
Rollback to an earlier Deployment revision if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
Scale up the Deployment to facilitate more load.
Pause the rollout of a Deployment to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
Use the status of the Deployment as an indicator that a rollout has stuck.
Clean up older ReplicaSets that you don't need anymore.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80



Note: The default output format for all kubectl commands is the human-readable plain-text format. The -o flag allows us to output the details in several different formats.

kubectl [command] [TYPE] [NAME] -o <output_format>

Here are some of the commonly used formats:
-o jsonOutput a JSON formatted API object.
-o namePrint only the resource name and nothing else.
-o wideOutput in the plain-text format with any additional information.
-o yamlOutput a YAML formatted API object.


NameSpaces: In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.)
Namespaces are a way to divide cluster resources between multiple users (via resource quota).
It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: use labels to distinguish resources within the same namespace.


What is dev.svc.cluster.local?
It's a fully qualified domain name (FQDN) used inside a Kubernetes cluster to resolve a service called dev running in the default namespace, using the internal Kubernetes DNS system.
FQDN Format	<service>.<namespace>.svc.cluster.local
