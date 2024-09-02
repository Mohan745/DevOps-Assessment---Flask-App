# Flask MongoDB Kubernetes Deployment

This tutorial will show you how to use Minikube to deploy a Flask application written in Python that is connected to a MongoDB database on a Kubernetes cluster. An API to input and retrieve data from MongoDB is offered by the Flask application. Kubernetes resources including Deployments, StatefulSets, Services, Persistent Volumes, and more will be used to deploy this configuration.

Make sure the following are installed on your system:

- Minikube
- Kubectl
- Docker
- Python 3.8 or later
- Pip (for managing Python packages)

## Steps to Deploy

### 1. Start Minikube

minikube start (This command will start a local Kubernetes cluster using Minikube)

### 2. Build and Push the Flask Docker Image

1. **Navigate to the project directory:**

   cd flask-mongodb-app

2. **Create a Dockerfile:**

   Dockerfile
```
FROM python:3.8-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
ENV FLASK_APP=app.py
CMD ["flask", "run", "--host=0.0.0.0"]
```

4. **Build the Docker image:**

   docker build -t flask-mongodb-app .

5. **Push the image to Docker Hub or a local registry (if required):**

   If using Docker Hub:

   docker tag flask-mongodb-app <your_dockerhub_username>/flask-mongodb-app
   docker push <your_dockerhub_username>/flask-mongodb-app
   
   If using Minikube's built-in Docker daemon:

   eval $(minikube docker-env)
   docker build -t flask-mongodb-app .

### 3. Create Kubernetes YAML Files

1. **Flask Deployment and Service:**

   Create a file named flask-deployment.yml

2. **MongoDB StatefulSet and Service:**

   Create a file named mongodb-statefulset.yml
   
### 4. Deploy to Minikube

1. **Apply the MongoDB StatefulSet:**

   kubectl apply -f mongodb-statefulset.yml

2. **Apply the Flask Deployment:**

   kubectl apply -f flask-deployment.yml

### 5. Access the Flask Application

1. **Get the NodePort of the Flask service:**

   kubectl get svc flask-service

2. **Access the application:**

   Open your browser and navigate to:

   http://<minikube_ip_address>:<NodePort>

   Replace `<minikube_ip_address>` with your IP address obtained from

### 6. Enable Autoscaling (Optional)

To set up Horizontal Pod Autoscaler (HPA) for the Flask application:

1. **Enable the metrics-server in Minikube:**

   ```bash
   minikube addons enable metrics-server
   ```

2. **Create HPA:**

   ```bash
   kubectl autoscale deployment flask-app --cpu-percent=70 --min=2 --max=5
   ```

### 7. Testing

- Use `curl` or any HTTP client to test the `/` and `/data` endpoints.
- Simulate high traffic to see the HPA in action.

### 8. Cleanup

To clean up the resources:

kubectl delete -f flask-deployment.yaml
kubectl delete -f mongodb-statefulset.yaml
minikube stop
minikube delete





Here's a detailed README to guide you through deploying the Flask application and MongoDB on a Minikube Kubernetes cluster.

---

# Flask MongoDB Kubernetes Deployment

## Overview

This guide will walk you through deploying a Python Flask application connected to a MongoDB database on a Kubernetes cluster using Minikube. The Flask application provides an API to insert and retrieve data from MongoDB, and we will deploy this setup using Kubernetes resources such as Deployments, StatefulSets, Services, Persistent Volumes, and more.

## Prerequisites

Ensure the following are installed on your system:

- Minikube
- Kubectl
- Docker
- Python 3.8 or later
- Pip (for managing Python packages)

## Steps to Deploy

### 1. Start Minikube

```bash
minikube start
```

This command will start a local Kubernetes cluster using Minikube.

### 2. Build and Push the Flask Docker Image

1. **Navigate to the project directory:**

   ```bash
   cd flask-mongodb-app
   ```

2. **Create a Dockerfile:**

   ```Dockerfile
   # Dockerfile
   FROM python:3.8-slim

   WORKDIR /app

   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt

   COPY . .

   ENV MONGODB_URI mongodb://mongodb-service:27017/

   CMD ["python", "app.py"]
   ```

3. **Build the Docker image:**

   ```bash
   docker build -t flask-mongodb-app .
   ```

4. **Push the image to Docker Hub or a local registry (if required):**

   If using Docker Hub:

   ```bash
   docker tag flask-mongodb-app <your_dockerhub_username>/flask-mongodb-app
   docker push <your_dockerhub_username>/flask-mongodb-app
   ```

   If using Minikube's built-in Docker daemon:

   ```bash
   eval $(minikube docker-env)
   docker build -t flask-mongodb-app .
   ```

### 3. Create Kubernetes YAML Files

1. **Flask Deployment and Service:**

   Create a file named `flask-deployment.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: flask-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: flask-app
     template:
       metadata:
         labels:
           app: flask-app
       spec:
         containers:
         - name: flask-app
           image: flask-mongodb-app:latest
           ports:
           - containerPort: 5000
           resources:
             requests:
               memory: "250Mi"
               cpu: "200m"
             limits:
               memory: "500Mi"
               cpu: "500m"
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-service
   spec:
     selector:
       app: flask-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 5000
     type: NodePort
   ```

2. **MongoDB StatefulSet and Service:**

   Create a file named `mongodb-statefulset.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: mongodb
   spec:
     serviceName: "mongodb-service"
     replicas: 1
     selector:
       matchLabels:
         app: mongodb
     template:
       metadata:
         labels:
           app: mongodb
       spec:
         containers:
         - name: mongodb
           image: mongo:latest
           ports:
           - containerPort: 27017
           env:
           - name: MONGO_INITDB_ROOT_USERNAME
             value: admin
           - name: MONGO_INITDB_ROOT_PASSWORD
             value: password
           volumeMounts:
           - name: mongo-persistent-storage
             mountPath: /data/db
     volumeClaimTemplates:
     - metadata:
         name: mongo-persistent-storage
       spec:
         accessModes: [ "ReadWriteOnce" ]
         resources:
           requests:
             storage: 1Gi
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: mongodb-service
   spec:
     ports:
       - port: 27017
     clusterIP: None
     selector:
       app: mongodb
   ```

### 4. Deploy to Minikube

1. **Apply the MongoDB StatefulSet:**

   ```bash
   kubectl apply -f mongodb-statefulset.yaml
   ```

2. **Apply the Flask Deployment:**

   ```bash
   kubectl apply -f flask-deployment.yaml
   ```

### 5. Access the Flask Application

1. **Get the NodePort of the Flask service:**

   ```bash
   kubectl get svc flask-service
   ```

   Look for the `NodePort` in the output, which will be something like `30001`.

2. **Access the application:**

   Open your browser and navigate to:

   ```
   http://<minikube_ip>:<NodePort>
   ```

   Replace `<minikube_ip>` with the IP address obtained from:

   ```bash
   minikube ip
   ```

   Replace `<NodePort>` with the port number you found in the previous step.

### 6. Enable Autoscaling (Optional)

To set up Horizontal Pod Autoscaler (HPA) for the Flask application:

1. **Enable the metrics-server in Minikube:**

   ```bash
   minikube addons enable metrics-server
   ```

2. **Create HPA:**

   ```bash
   kubectl autoscale deployment flask-app --cpu-percent=70 --min=2 --max=5
   ```

### 7. Testing

- Use `curl` or any HTTP client to test the `/` and `/data` endpoints.
- Simulate high traffic to see the HPA in action.

### 8. Cleanup

To clean up the resources:

```bash
kubectl delete -f flask-deployment.yaml
kubectl delete -f mongodb-statefulset.yaml
minikube stop
minikube delete
```

## DNS Resolution Explanation

In Kubernetes, DNS resolution is handled by `kube-dns` or `CoreDNS`, which provides internal DNS services for the cluster. Each Service in Kubernetes is assigned a DNS name in the format `service-name.namespace.svc.cluster.local`. When the Flask application tries to connect to MongoDB using `mongodb-service`, Kubernetes resolves this name to the corresponding Pod IPs within the cluster.

## Resource Requests and Limits Explanation

In Kubernetes, resource requests and limits ensure that each container gets the required amount of CPU and memory. The `requests` specify the minimum resources the container is guaranteed, while `limits` define the maximum resources it can use. This helps in optimizing resource allocation and avoiding scenarios where a container starves others by consuming excessive resources.

---

This README should guide you through the entire process of deploying your Flask application with MongoDB on a Kubernetes cluster using Minikube.
