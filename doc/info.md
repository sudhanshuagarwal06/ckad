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

![Master and Worker Nodes](master_wroker_node.PNG)

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
```

## Deployment

A Deployment manages a set of Pods to run an application workload, usually one that doesn't maintain state. 

### Use Cases for Deployments

Typical use cases for Deployments include:

- **Rollout a ReplicaSet**: Create a Deployment to rollout a ReplicaSet. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
- **Update Pods**: Declare the new state of the Pods by updating the `PodTemplateSpec` of the Deployment. A new ReplicaSet is created, and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
- **Rollback**: Rollback to an earlier Deployment revision if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
- **Scale Up**: Scale up the Deployment to facilitate more load.
- **Pause and Resume Rollout**: Pause the rollout of a Deployment to apply multiple fixes to its `PodTemplateSpec` and then resume it to start a new rollout.
- **Monitor Rollout Status**: Use the status of the Deployment as an indicator that a rollout has stuck.
- **Cleanup**: Clean up older ReplicaSets that you don't need anymore.

### Example Deployment Manifest

```yaml
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
```

## Namespaces
In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g., Deployments, Services) and not for cluster-wide objects (e.g., StorageClass, Nodes, PersistentVolumes).

Namespaces are a way to divide cluster resources between multiple users (via resource quota). It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software; use labels to distinguish resources within the same namespace.

## Overriding Container Commands
When you create a Pod in Kubernetes, you can define a command and arguments for the container(s) to override the default `ENTRYPOINT` and `CMD` specified in the container image.

### Example Pod Manifest with Command and Arguments
```yaml
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
```

Breakdown:

- **command**: This overrides the `ENTRYPOINT` in the container image. In this case, it's `["sh", "-c"]`, telling the container to run a shell command.
- **args**: These are the arguments passed to the command. Here, it's a single shell command: `echo Hello from the Pod! && sleep 3600`.
- If `command` is omitted, the image’s default `ENTRYPOINT` is used. If `args` is omitted, the image’s default `CMD` is used.

## Defining Environment Variables for Containers
When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the `env` or `envFrom` field in the configuration file. The `env` and `envFrom` fields have different effects:

- **env**: Allows you to set environment variables for a container, specifying a value directly for each variable that you name.

### Example Pod Manifest with Environment Variables

```yaml
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
```

- **envFrom**: Allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use `envFrom`, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string.
  
### Example Pod Manifest Using envFrom

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo-from-configmap
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/hello-app:2.0
    envFrom:
    - configMapRef:
        name: my-config
    - secretRef:
        name: my-secret
```

In this example, all key-value pairs from the `my-config` ConfigMap and `my-secret` Secret will be set as environment variables in the `envar-demo-container`.

## ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

### Creating a ConfigMap from Literal Values

You can create a ConfigMap directly from the command line using literal values:

```bash
kubectl create configmap <configmap_name> --from-literal=<key>=<value> --from-literal=<key>=<value>
```

### Creating a ConfigMap from a File
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  database_host: "192.168.0.1"
  debug_mode: "1"
  log_level: "verbose"
```

### Using ConfigMap in a Pod
```yaml
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
```

### Example of Using ConfigMap Key Reference
```yaml
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
```

## Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Using a Secret means that you don't need to include confidential data in your application code.

### Creating a Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: my-awesome-password
```

You can also create a Secret from the command line:

```bash
kubectl create secret generic database-credentials \
--from-literal=username=username \
--from-literal=password=password
```

## Types of Kubernetes Secrets

Kubernetes supports several types of secrets:

- **Service Account Token Secrets: Stores a token credential that identifies a service account.
- **Docker Config Secrets**: Stores credentials for accessing a container image registry.
- **Basic Authentication Secrets**: Stores credentials needed for basic authentication.
- **SSH Authentication Secrets**: Stores data used in SSH authentication.
- **TLS Secrets**: Used to store a certificate and its associated key.
- **Bootstrap Token Secrets**: Stores bootstrap token data during the node bootstrap process.

## Configure a Security Context for a Pod or Container

A security context defines privilege and access control settings for a Pod or Container.

### Example Pod with Security Context

```yaml
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
```

## Adding Capabilities
Here is a configuration file for a Pod that runs one Container and adds the NET_ADMIN and SYS_TIME capabilities:

```yaml
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
```

## What Are Service Accounts?

A service account is a type of non-human account that provides a distinct identity in a Kubernetes cluster. This identity is useful for authenticating to the API server or implementing identity-based security policies.

### Default Service Accounts

Kubernetes automatically creates a ServiceAccount object named `default` for every namespace in your cluster. If you deploy a Pod in a namespace without manually assigning a ServiceAccount, Kubernetes assigns the default ServiceAccount for that namespace.

### Use Cases for Kubernetes Service Accounts

Service accounts can be used in scenarios such as:

- Communicating with the Kubernetes API server.
- Authenticating to a private image registry using an imagePullSecret.
- Allowing external services to communicate with the Kubernetes API server.

Practical Implementation

1. **Creating Service Accounts**: Use the Kubernetes command-line interface (kubectl) to create Service Accounts within the cluster, specifying metadata such as labels or annotations if needed.

```bash
kubectl create serviceaccount <serviceaccount_name>
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: my-namespace
Creating Service Accounts:
```

2. **Granting Permissions**: Define roles and role bindings to grant permissions to Service Accounts, specifying the actions and resources they are allowed to access within the cluster.

### ROLE :

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
    namespace: default
    name: service-account-role
rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "create", "delete", list"]
```

### ROLE-BINDING :

```yaml
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
```

3. **Associating Service Accounts with Pods**: Configure pods to use specific Service Accounts by referencing their names in pod specifications or deployment manifests.

### POD:

```yaml
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
```

4. ***Testing Authentication and Authorization***: Verify that pods can authenticate with the Kubernetes API server using their Service Accounts and that they have the necessary permissions to perform desired operations within the cluster.

### Generate toke for serviceaccount

```bash
kubectl create token <serviceaccount_name>
```

## Resource Management for Pods and Containers
When you specify a Pod, you can optionally specify how much of each resource a container needs. The most common resources to specify are CPU and memory (RAM).

When you specify the resource request for containers in a Pod, the kube-scheduler uses this information to decide which node to place the Pod on. The kubelet enforces resource limits so that the running container does not exceed the specified limits.

### Example Pod with Resource Requests and Limits

```yaml
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
        - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: "1"
        limits:
          memory: "2Gi"
          cpu: "2"
```

## Taints and Tolerations
Taints allow a node to repel a set of Pods, while tolerations are applied to Pods to allow the scheduler to schedule them on nodes with matching taints. Taints and tolerations work together to ensure that Pods are not scheduled onto inappropriate nodes.

### Taint Types
- **NoExecute**: Affects Pods that are already running on the node. Pods that do not tolerate the taint are evicted immediately.
- **NoSchedule**: No new Pods will be scheduled on the tainted node unless they have a matching toleration.
- **PreferNoSchedule**: A "soft" version of NoSchedule; the control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.

### Example of Tainting a Node
```bash
kubectl taint nodes node1 key1=value1:NoSchedule
```
### Example Pod with Tolerations
```yaml
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
```

### Untainting a Node
To remove a taint from a node, use the following command:
```bash
kubectl taint nodes <node-name> <taint-key>=<taint-value>:<taint-effect>-
```



