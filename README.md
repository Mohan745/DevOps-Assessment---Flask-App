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

3. **Build the Docker image:**

   docker build -t flask-mongodb-app .

4. **Push the image to Docker Hub or a local registry (if required):**

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
