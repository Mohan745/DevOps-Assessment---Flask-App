Flask MongoDB Kubernetes Deployment
Overview
This guide will walk you through deploying a Python Flask application connected to a MongoDB database on a Kubernetes cluster using Minikube. The Flask application provides an API to insert and retrieve data from MongoDB, and we will deploy this setup using Kubernetes resources such as Deployments, StatefulSets, Services, Persistent Volumes, and more.

Prerequisites
Ensure the following are installed on your system:

Minikube
Kubectl
Docker
Python 3.8 or later
Pip (for managing Python packages)
Steps to Deploy
1. Start Minikube
bash
Copy code
minikube start
This command will start a local Kubernetes cluster using Minikube.

2. Build and Push the Flask Docker Image
Navigate to the project directory:

bash
Copy code
cd flask-mongodb-app
Create a Dockerfile:

Dockerfile
Copy code
# Dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

ENV MONGODB_URI mongodb://mongodb-service:27017/

CMD ["python", "app.py"]
Build the Docker image:

bash
Copy code
docker build -t flask-mongodb-app .
Push the image to Docker Hub or a local registry (if required):

If using Docker Hub:

bash
Copy code
docker tag flask-mongodb-app <your_dockerhub_username>/flask-mongodb-app
docker push <your_dockerhub_username>/flask-mongodb-app
If using Minikube's built-in Docker daemon:

bash
Copy code
eval $(minikube docker-env)
docker build -t flask-mongodb-app .
3. Create Kubernetes YAML Files
Flask Deployment and Service:

Create a file named flask-deployment.yaml:

yaml
Copy code
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
MongoDB StatefulSet and Service:

Create a file named mongodb-statefulset.yaml:

yaml
Copy code
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
4. Deploy to Minikube
Apply the MongoDB StatefulSet:

bash
Copy code
kubectl apply -f mongodb-statefulset.yaml
Apply the Flask Deployment:

bash
Copy code
kubectl apply -f flask-deployment.yaml
5. Access the Flask Application
Get the NodePort of the Flask service:

bash
Copy code
kubectl get svc flask-service
Look for the NodePort in the output, which will be something like 30001.

Access the application:

Open your browser and navigate to:

php
Copy code
http://<minikube_ip>:<NodePort>
Replace <minikube_ip> with the IP address obtained from:

bash
Copy code
minikube ip
Replace <NodePort> with the port number you found in the previous step.

6. Enable Autoscaling (Optional)
To set up Horizontal Pod Autoscaler (HPA) for the Flask application:

Enable the metrics-server in Minikube:

bash
Copy code
minikube addons enable metrics-server
Create HPA:

bash
Copy code
kubectl autoscale deployment flask-app --cpu-percent=70 --min=2 --max=5
7. Testing
Use curl or any HTTP client to test the / and /data endpoints.
Simulate high traffic to see the HPA in action.
