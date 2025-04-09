# Kubernetes Components

## Control Plane Components

The control plane manages the overall state of the cluster and consists of the following components:

- **kube-apiserver**: The core component server that exposes the Kubernetes HTTP API.
- **etcd**: A consistent and highly-available key-value store for all API server data.
- **kube-scheduler**: Looks for Pods that are not yet bound to a node and assigns each Pod to a suitable node.
- **kube-controller-manager**: Runs controllers to implement Kubernetes API behavior.
- **cloud-controller-manager (optional)**: Integrates with underlying cloud provider(s).

## Node Components

Node components run on every node, maintaining running Pods and providing the Kubernetes runtime environment:

- **kubelet**: Ensures that Pods are running, including their containers.
- **kube-proxy (optional)**: Maintains network rules on nodes to implement Services.
- **Container runtime**: Software responsible for running containers. 

### Nodes

Kubernetes runs your workload by placing containers into Pods to run on Nodes. A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods.

A Kubernetes cluster consists of two types of nodes:
- **Master Node**: Hosts the Kubernetes control plane and manages the cluster, including scheduling and scaling applications and maintaining the state of the cluster.
- **Worker Nodes**: Responsible for running the containers and executing the workloads.

![Master and Worker Nodes](master_worker_node.PNG)

## What Is A Kubernetes Cluster?

Nodes usually work together in groups. A Kubernetes cluster contains a set of work machines (nodes) that automatically distribute workload among themselves, enabling seamless scaling.

## Command Line Tool (kubectl)

Kubernetes provides a command line tool for communicating with a Kubernetes cluster's control plane, using the Kubernetes API.

### Syntax

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

Where:
- **command**: Specifies the operation you want to perform on one or more resources (e.g., `create`, `get`, `describe`, `delete`).
- **TYPE**: Specifies the resource type (case-insensitive; can be singular, plural, or abbreviated).
- **NAME**: Specifies the name of the resource (case-sensitive). If omitted, details for all resources are displayed (e.g., `kubectl get pods`).
- **flags**: Specifies optional flags (e.g., `-s` or `--server` to specify the address and port of the Kubernetes API server).

## What Is A Kubernetes Pod?

A Kubernetes Pod is a collection of one or more application containers. The Pod provides shared storage (volumes), an IP address, communication between containers, and hosts other information about how to run application containers. Thus, containers do not run directly on virtual machines; Pods are a way to manage containers.

### Example Pod Manifest

```yaml
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
```

## Replication Controller
A ReplicationController ensures that a specified number of Pod replicas are running at any one time, making sure that a Pod or a homogeneous set of Pods is always up and available.

### Example Replication Controller Manifest

```yaml
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
```

### ReplicaSet

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time, often used to guarantee the availability of a specified number of identical Pods.

## Scale the ReplicaSet
To scale the ReplicaSet, use the following command:

```bash
kubectl scale replicaset <replicaset_name> --replicas=<desired_number>
```

Example ReplicaSet Manifest

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3  # Modify replicas according to your case
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
```

### Comparison: ReplicaSet vs Replication Controller

|                   Replica Set                    |               Replication Controller                          |
|--------------------------------------------------|---------------------------------------------------------------|
| Replica Set supports the new set-based selector. | Replication Controller only supports equality-based selector. |
| This gives more flexibility. For example:        | For example:                                                  |
| Environment in (production, qa)                  | Environment = production                                      |
| This selects all resources with key equal to     | This selects all resources with key equal to                  |
| environment and value equal to production or qa  | environment and value equal to production                     |

## Labels and Selectors

Labels are key/value pairs that are attached to objects such as Pods. They are intended to specify identifying attributes of objects that are meaningful and relevant to users but do not directly imply semantics to the core system. Labels can be used to organize and select subsets of objects. 

- Labels can be attached to objects at creation time and subsequently added and modified at any time.
- Each object can have a set of key/value labels defined, and each key must be unique for a given object.

### Example of Metadata with Labels

```yaml
metadata:
  labels:
    key1: value1
    key2: value2





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



Note: When you create a Pod in Kubernetes, you can define a command and arguments for the container(s) to override the default ENTRYPOINT and CMD specified in the container image.
Here’s how you do it in a Pod manifest YAML:

apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ["sh", "-c"]
      args: ["echo Hello from the Pod! && sleep 3600"]

Breakdown:
command: This overrides the ENTRYPOINT in the container image. In this case, it's ["sh", "-c"], telling the container to run a shell command.
args: These are the arguments passed to the command. Here, it's a single shell command: echo Hello from the Pod! && sleep 3600.
If command is omitted, the image’s default ENTRYPOINT is used. If args is omitted, the image’s default CMD is used.


Define an environment variable for a container
When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the env or envFrom field in the configuration file. The env and envFrom fields have different effects.

env: allows you to set environment variables for a container, specifying a value directly for each variable that you name.

apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/hello-app:2.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"

envFrom: allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use envFrom, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string.


ConfigMaps: A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

You can create a ConfigMap directly from the command line using literal values.
kubectl create configmap <configmap_name> --from-literal=<key>=<value> --from-literal=<key>=<value>

Creating a ConfigMap from a File

apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  database_host: "192.168.0.1"
  debug_mode: "1"
  log_level: "verbose"

apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: demo-config

apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: CONFIGMAP_USERNAME
      valueFrom:
        configMapKeyRef:
          name: myconfigmap
          key: username


Secrets: A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.

Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods. Kubernetes, and applications that run in your cluster, can also take additional precautions with Secrets, such as avoiding writing sensitive data to nonvolatile storage.

apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: my-awesome-password

 kubectl create secret generic database-credentials \
--from-literal=username=username \
--from-literal=password=password

Types of Kubernetes Secrets
Kubernetes supports several types of secrets:

Opaque Secrets: Opaque Secrets are used to store arbitrary user-defined data. Opaque is the default Secret type, meaning that when you don’t specify any type when creating a Secret, the secret will be considered Opaque.
Service account token Secrets: This Secret type stores a token credential that identifies a service account. It is important to note that when using this Secret type, you must ensure that the kubernetes.io/service-account.name annotation is set to an existing service account name.
Docker config Secrets: Docker config secret stores the credentials for accessing a container image registry. You use Docker config secret with one of the following type values:
kubernetes.io/dockercfg
kubernetes.io/dockerconfigjson
Basic authentication Secret: Basic authentication type stores credentials needed for basic authentication.  When using this type of Secret, the data field must contain at least one of the following keys:
username: the user name for authentication
password: the password or token for authentication
SSH authentication secrets: This Secret type stores data used in SSH authentication. When using an SSH authentication, you must specify a ssh-privatekey key-value pair in the data (or stringData) field as the SSH credential to use.
TLS Secrets: You use this Secret type to store a certificate and its associated key typically used for TLS. When using a TLS secret, you must provide the tls.key and the tls.crt key in the configuration’s data (or stringData) field. 
Bootstrap token Secrets: You use this Secret type to store bootstrap token data during the node bootstrap process. You typically create a bootstrap token Secret in the kube-system namespace and named it in the form bootstrap-token-<token-id>.


Configure a Security Context for a Pod or Container: A security context defines privilege and access control settings for a Pod or Container
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper

Here is the configuration file for a Pod that runs one Container. The configuration adds the NET_ADMIN and SYS_TIME capabilities:

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]


What are service accounts?
A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster. Application Pods, system components, and entities inside and outside the cluster can use a specific ServiceAccount's credentials to identify as that ServiceAccount. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies.
Default service accounts
When you create a cluster, Kubernetes automatically creates a ServiceAccount object named default for every namespace in your cluster. The default service accounts in each namespace get no permissions by default other than the default API discovery permissions that Kubernetes grants to all authenticated principals if role-based access control (RBAC) is enabled. If you delete the default ServiceAccount object in a namespace, the control plane replaces it with a new one.

If you deploy a Pod in a namespace, and you don't manually assign a ServiceAccount to the Pod, Kubernetes assigns the default ServiceAccount for that namespace to the Pod.

Use cases for Kubernetes service accounts
As a general guideline, you can use service accounts to provide identities in the following scenarios:

Your Pods need to communicate with the Kubernetes API server, for example in situations such as the following:
Providing read-only access to sensitive information stored in Secrets.
Granting cross-namespace access, such as allowing a Pod in namespace example to read, list, and watch for Lease objects in the kube-node-lease namespace.
Your Pods need to communicate with an external service. For example, a workload Pod requires an identity for a commercially available cloud API, and the commercial provider allows configuring a suitable trust relationship.
Authenticating to a private image registry using an imagePullSecret.
An external service needs to communicate with the Kubernetes API server. For example, authenticating to the cluster as part of a CI/CD pipeline.
You use third-party security software in your cluster that relies on the ServiceAccount identity of different Pods to group those Pods into different contexts

Practical Implementation:
In a hands-on demonstration, let’s walk through the process of creating and using Service Accounts in a Kubernetes cluster:

1. Creating Service Accounts: Use the Kubernetes command-line interface (kubectl) to create Service Accounts within the cluster, specifying metadata such as labels or annotations if needed.
kubectl create serviceaccount <serviceaccount_name>

apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: my-namespace

2. Granting Permissions: Define roles and role bindings to grant permissions to Service Accounts, specifying the actions and resources they are allowed to access within the cluster.

ROLE :
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
    namespace: default
    name: service-account-role
rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "create", "delete", list"]


ROLE-BINDING :
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: mysa-rolebinding
    namespace: default
subjects:
    - kind: ServiceAccount
      name: my-sa
      apiGroup: ""
roleRef:
    kind: Role
    name: service-account-role
    apiGroup: rbac.authorization.k8s.io

3. Associating Service Accounts with Pods: Configure pods to use specific Service Accounts by referencing their names in pod specifications or deployment manifests.

POD:
apiVersion: v1
kind: Pod
metadata: 
    name: 'my-pod'
    namespace: default
spec:
    containers:
      - name: first-container
        image: vimal13/apache-webserver-php
    serviceAccountName: my-sa

4. Testing Authentication and Authorization: Verify that pods can authenticate with the Kubernetes API server using their Service Accounts and that they have the necessary permissions to perform desired operations within the cluster.

Generate toke for serviceaccount
kubectl create token <serviceaccount_name>


Resource Management for Pods and Containers: When you specify a Pod, you can optionally specify how much of each resource a container needs. The most common resources to specify are CPU and memory (RAM); there are others.

When you specify the resource request for containers in a Pod, the kube-scheduler uses this information to decide which node to place the Pod on. When you specify a resource limit for a container, the kubelet enforces those limits so that the running container is not allowed to use more of that resource than the limit you set. The kubelet also reserves at least the request amount of that system resource specifically for that container to use.
CPU and memory are each a resource type. A resource type has a base unit. CPU represents compute processing and is specified in units of Kubernetes CPUs. Memory is specified in units of bytes


apiVersion: v1
kind: Pod
metadata:
  name: simple-app
  labels:
    name: simple-app
spec:
 containers:
 - name: simple-app
   image: simple-app
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
     limits:
       memory: "2Gi"
       cpu: "2"



Taints and Tolerations: 
Taints  allow a node to repel a set of pods.
Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.
Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

NoExecute: This affects pods that are already running on the node as follows:
    Pods that do not tolerate the taint are evicted immediately
    Pods that tolerate the taint without specifying tolerationSeconds in their toleration specification remain bound forever
    Pods that tolerate the taint with a specified tolerationSeconds remain bound for the specified amount of time. After that time elapses, the node lifecycle controller evicts the Pods from the node.
NoSchedule: No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.
PreferNoSchedule: PreferNoSchedule is a "preference" or "soft" version of NoSchedule. The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.

kubectl taint nodes node1 key1=value1:NoSchedule

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
    - key: "key1"
      operator: "Equal"
      value: "value1"
      effect: "NoSchedule"

to untaint a node: kubectl taint nodes <node-name> <taint-key>=<taint-value>:<taint-effect>-
