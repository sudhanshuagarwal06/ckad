# Multi-Container-POD, POD-Observability & POD-Design

## Single Container Pods
This is the most common use case where a Pod wraps a single container. Kubernetes manages the lifecycle of the Pod rather than the individual container.

## Multi-Container Pods
In some scenarios, you may have multiple tightly-coupled containers that need to work together. For example, one container might serve data stored in a shared volume while another container (often referred to as a sidecar) refreshes or updates those files. The Pod encapsulates these containers, their storage resources, and provides an ephemeral network identity as a single unit.

## Init Containers
In a multi-container Pod, each container is expected to run a process that remains active for the entire lifecycle of the Pod. For instance, in a Pod containing a web application and a logging agent, both containers should stay alive as long as the web application is running. If either container fails, the Pod will restart.

However, there are scenarios where you may want to run a process that completes and exits, such as:
- Pulling code or binaries from a repository for use by the main application.
- Waiting for an external service or database to become available before starting the main application.
This is where Init Containers come into play. Init Containers are specialized containers that run to completion before the main application containers start.

### Example Pod Configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: examplePod
spec:
  initContainers:
  - name: init-myservice
    image: nginx
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: nginx
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
  containers:
  - name: myapp-container
    image: nginx
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

By using Init Containers, you can ensure that necessary preconditions are met before your main application starts, leading to a more robust and reliable deployment.


## Liveness Probe
A liveness probe checks whether a container is still running correctly. If the liveness probe fails, Kubernetes restarts the container to recover from potential deadlocks, crashes, or unresponsive states that the application cannot fix on its own.

Use liveness probes to:
- Detect and recover from application hangs or deadlocks
- Automatically restart faulty containers

### Example Liveness Probe
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: examplePod
spec:
  containers:
  - name: nginx
    image: nginx
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Readiness Probe
A readiness probe checks whether a container is ready to serve requests. Unlike a liveness probe, failing a readiness probe does not restart the container. Instead, Kubernetes will temporarily remove the Pod from the Service's endpoints, preventing traffic from being routed to it.

Use readiness probes when:
- The application needs time during startup (e.g., loading large files or dependencies)
- External services must be available before the app can respond to traffic

### Example Readiness Probe
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: examplePod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 2379
   readinessProbe:
    httpGet:
        path: /ready
        port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
```

### Types of Probes
Kubernetes supports three types of health checks for both liveness and readiness probes:

1. **HTTP GET Probe**: Sends an HTTP GET request to a specified endpoint. The probe succeeds if the response status is 2xx or 3xx.
```yaml
readinessProbe:
  httpGet:
    path: /testpath
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```
2. **TCP Socket Probe**: Attempts to open a TCP connection on a specific port. The probe passes if the connection is successful. This is useful for non-HTTP applications.
```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```
3. **Exec (Command) Probe**: Runs a command inside the container. If the command exits with status 0, the probe is considered successful.
```yaml
readinessProbe:
  exec:
    command:
      - /bin/sh
      - -c
      - check-script.sh
  initialDelaySeconds: 20
  periodSeconds: 15
```

## Kubernetes Metrics
Kubernetes metrics provide essential insights into the performance and health of your clusters, nodes, pods, and applications. These metrics help DevOps teams and IT operations monitor system behavior, detect anomalies, and take proactive action before issues impact users.

**Cluster Visibility** – Get a real-time overview of resource usage (CPU, memory, network, etc.)
**Performance Monitoring** – Identify performance bottlenecks and slowdowns
**Error Detection** – Quickly catch failing pods, misbehaving nodes, or overutilized resources
**Scaling Decisions** – Drive autoscaling based on actual resource usage
**Reliability** – Ensure your applications are running efficiently and resiliently

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

## Kubernetes Annotations
Kubernetes annotations are key-value metadata pairs that you can attach to Kubernetes objects such as Pods, ReplicaSets, Deployments, and more. They allow you to store non-identifying information that helps tools, libraries, and automation workflows operate more intelligently.

### Example of Metadata with Annotations
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

