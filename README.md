<h1 align=center> minikube project guide </h1>

-------
- This initializes a Minikube cluster running within a Docker container. Afterward, you can interact with the Kubernetes cluster using kubectl or other Kubernetes tools.
```bash
minikube start --driver docker # is used to start a local Kubernetes cluster using Minikube with the Docker driver.
minikube status
kubectl get node # to get the nodes
```

- create `mongo-config.yaml`
- create `mongo-secret.yaml`
```bash
# This command encodes the string mongouser into Base64 format
echo -n mongouser | base64 # for linux and mac or On Windows with MinGW, Git Bash, or Cygwin
[convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("mongouser")) # powerShell
echo -n mongopassword | base64
```
- create `mongo.yaml`

- create `webapp.yaml`
  - go to docker hub and create an image
  
```bash
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml


kubectl get all
kubectl get secret
kubectl get pod

kubectl get svc

```

- It is required to pull the app from the docker hub

`Resources:` https://gitlab.com/nanuchi/k8s-in-1-hour

-------
-------
-------
-------

**Kubernetes** (often abbreviated as K8s) is an open-source platform for managing containerized applications across multiple hosts. It provides mechanisms for deploying, scaling, and operating application containers. Below is a comprehensive overview, including practical examples, covering its key components and functionality.

## Core Concepts of Kubernetes
1. Kubernetes Architecture
Master Node (Control Plane):

API Server: Entry point for managing the cluster, communicates via REST API.
Controller Manager: Handles controllers like replication, node, endpoint, and namespace.
Scheduler: Assigns workloads to nodes based on resource availability.
etcd: A distributed key-value store for cluster data and configuration.
Worker Node:

Kubelet: Ensures containers are running in a Pod.
Kube-Proxy: Maintains network rules and connectivity.
Container Runtime: Runs the containers, e.g., Docker, containerd.

2. Key Kubernetes Objects
Pod: Smallest deployable unit, typically one or more containers.
Service: Exposes a Pod or a set of Pods to network traffic.
Deployment: Manages Pod lifecycle and ensures a specified state.
ConfigMap & Secret: Stores configuration data and sensitive information.
Persistent Volume (PV) & Persistent Volume Claim (PVC): Manage storage.

3. Kubernetes Features
Scaling: Automatic or manual scaling of containers.
Self-Healing: Restarts failed containers, reschedules Pods.
Load Balancing: Distributes traffic across Pods.
Rolling Updates and Rollbacks: Ensures smooth application updates.
Namespaces: Provides isolation and management of cluster resources.

## Step-by-Step Practical Example
### 1. Setting Up Kubernetes Cluster
Use a cloud provider like AWS (EKS), Azure (AKS), Google Cloud (GKE), or local tools like Minikube or Kind.

Minikube Installation
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube
minikube start
```

### 2. Deploy a Simple Application
2.1 Create a Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.21.6
        ports:
        - containerPort: 80
```

Apply the configuration:
```bash
kubectl apply -f deployment.yaml
```

2.2 Expose Deployment with a Service
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
``` 
Apply the configuration:

```bash
kubectl apply -f service.yaml
```

Check the service:
```bash
kubectl get svc
```
Access the application using the service's NodePort.

### 3. Scaling the Application
Increase replicas for the deployment:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### 4. Update the Application
Update the nginx image to a new version:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.23.1
```

Rollback if needed:

```bash
kubectl rollout undo deployment/nginx-deployment
```

### 5. Use ConfigMap and Secret
5.1 Create a ConfigMap
```bash
kubectl create configmap app-config --from-literal=APP_ENV=production
```
5.2 Use ConfigMap in a Pod
```yaml
# pod-with-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "echo $(APP_ENV) && sleep 3600" ]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
```

Apply the configuration:
```bash
kubectl apply -f pod-with-configmap.yaml
```

### 6. Persistent Storage
6.1 Define a Persistent Volume
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```
6.2 Create a Persistent Volume Claim
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
### 7. Observability and Monitoring
7.1 View Logs
```bash
kubectl logs <pod-name>
```
7.2 View Events
```bash
kubectl get events
```
7.3 Use Metrics Server
Install the metrics server to view resource utilization:

```bash
kubectl top pods
```
8. Kubernetes Best Practices
- Use Namespaces to separate environments (e.g., dev, staging, production).
- Apply RBAC (Role-Based Access Control) for fine-grained permissions.
- Use Helm Charts to package applications.
- Implement Pod Affinity/Anti-Affinity for better placement.
- Monitor with tools like Prometheus, Grafana, or Kubernetes Dashboard.