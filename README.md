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

```bash
minikube start (This command will start a local Kubernetes cluster using Minikube)
```

### 2. Build and Push the Flask Docker Image

1. **Navigate to the project directory:**

```bash
   cd flask-mongodb-app
```

3. **Create a Dockerfile:**

```Dockerfile
   FROM python:3.8-slim
   WORKDIR /app
   COPY . /app
   RUN pip install --no-cache-dir -r requirements.txt
   EXPOSE 5000
   ENV FLASK_APP=app.py
   CMD ["flask", "run", "--host=0.0.0.0"]
```

4. **Build the Docker image:**
   
```
   docker build -t flask-mongodb-app .
```

5. **Push the image to Docker Hub or a local registry (if required):**

   If using Docker Hub:
   
```
   docker tag flask-mongodb-app <your_dockerhub_username>/flask-mongodb-app
   docker push <your_dockerhub_username>/flask-mongodb-app
```

   If using Minikube's built-in Docker daemon:

```
   eval $(minikube docker-env)
   docker build -t flask-mongodb-app .
```

### 3. Create Kubernetes YAML Files

1. **Flask Deployment and Service:**

   Create a file `flask_deployment.yml`

```apiVersion: apps/v1
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
        image: your-dockerhub-username/flask-mongodb-app:latest
        ports:
        - containerPort: 5000
        env:
        - name: MONGODB_URI
          value: "mongodb://mongo:27017/"
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
  type: NodePort
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007
```   

3. **MongoDB StatefulSet and Service:**

   Create a file named `mongodb_statefulset.yml`
```
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 1
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "Mohan"
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: "Mohan745"
  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
    spec:
      accessModes:
      resources:
        requests:
          storage: 1Gi
```
   
### 4. Deploy to Minikube

1. **Apply the MongoDB StatefulSet:**

```
   kubectl apply -f mongodb_statefulset.yml
```

2. **Apply the Flask Deployment:**

```
   kubectl apply -f flask_deployment.yml
```

### 5. Access the Flask Application

1. **Get the NodePort of the Flask service:**

```
   kubectl get svc flask-service
```

2. **Access the application:**

   Open your browser and navigate to:
```
   http://<minikube_ip_address>:<NodePort>
```
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
```
kubectl delete -f flask_deployment.yml
kubectl delete -f mongodb_statefulset.yml
minikube stop
minikube delete
```
