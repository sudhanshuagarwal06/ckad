# Kubernetes Service Naming and Imperative Commands

## Understanding Kubernetes Service Naming

In a Kubernetes cluster, services are identified using fully qualified domain names (FQDNs). For example, the FQDN `dev.svc.cluster.local` refers to a service named `dev` that is running in the `default` namespace. The general format for an FQDN in Kubernetes is: <service>.<namespace>.svc.cluster.local

This naming convention allows for easy service discovery within the cluster using the internal Kubernetes DNS system.

## Imperative Commands in Kubernetes

While most interactions with Kubernetes are done declaratively through resource definition files, imperative commands can be very useful for quick, one-time tasks and for generating templates for resource definitions. This can save you significant time, especially during exams or when you need to prototype quickly.

### Helpful Options

When using imperative commands, two options can enhance your workflow:

- `--dry-run=client`: This option allows you to test your command without actually creating the resource. It verifies whether the resource can be created and checks the correctness of your command.
  
- `-o yaml`: This option outputs the resource definition in YAML format, which is the standard format for Kubernetes resource definitions.

### Generating Resource Definitions

You can combine these options with Linux output redirection to quickly generate a resource definition file. This approach allows you to create a template that you can modify as needed, rather than starting from scratch.

### Example Command

To create a YAML definition for an Nginx pod without actually deploying it, you can use the following command:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

This command will generate a file named nginx-pod.yaml containing the definition of an Nginx pod, which you can then edit and apply as required.

# Kubernetes Pod, Deployment, and Service Management

## Pod Management

### Create an NGINX Pod

To create a simple NGINX Pod, use the following command:

```bash
kubectl run nginx --image=nginx
```

### Generate Pod Manifest YAML File
To generate a Pod manifest YAML file without creating it, use the --dry-run option:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

## Deployment Management

### Create a Deployment

To create a deployment for NGINX, use the following command:

```bash
kubectl create deployment --image=nginx nginx
```

### Generate Deployment YAML File

To generate a deployment manifest YAML file without creating it, use the following command:

```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

### Generate Deployment with 4 Replicas

To create a deployment with 4 replicas, use the following command:

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

### Scale Deployment

You can scale an existing deployment using the kubectl scale command:

```bash
kubectl scale deployment nginx --replicas=4
```

### Save YAML Definition to File

Another way to create a deployment is to save the YAML definition to a file, modify it, and then create the deployment:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

You can then update the YAML file with the desired number of replicas or any other fields before creating the deployment.

## Service Management

### Create a ClusterIP Service

To create a Service named redis-service of type ClusterIP to expose the Pod redis on port 6379, use the following command:

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
(This will automatically use the Pod's labels as selectors.)

Alternatively, you can use:

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```
(This will not use the Pod's labels as selectors; instead, it will assume selectors as app=redis. You cannot pass in selectors as an option, so it may not work well if your Pod has a different label set. Generate the file and modify the selectors before creating the service.)

### Create a NodePort Service

To create a Service named nginx of type NodePort to expose the Pod nginx's port 80 on port 30080 on the nodes, use the following command:

```bash
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```
(This will automatically use the Pod's labels as selectors, but you cannot specify the node port. You must generate a definition file and then add the node port manually before creating the service.)

Alternatively, you can use:

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
(This will not use the Pod's labels as selectors.)


Recommendations
Both the above commands have their own challenges. While one cannot accept a selector, the other cannot accept a node port. It is recommended to use the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the node port before creating the service.
