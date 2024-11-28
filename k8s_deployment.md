# Deploying a Kubernetes Cluster with a Custom Docker Image

This guide includes building a Docker image, pushing it to a registry, and deploying the containerized application on a Kubernetes cluster.

## Step 1: Build and Push a Docker Image

### 1.1. Create a Simple Application

Create a simple Node.js app.

```bash
mkdir my-app && cd my-app
```

Create a file named app.js:

```javascript
// app.js
const http = require('http');
const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, Kubernetes!\n');
});

server.listen(port, () => {
  console.log(`Server running at http://localhost:${port}/`);
});
```

Create a package.json file:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A simple Node.js app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {}
}
```

Install dependencies:

```bash
npm install
```

### 1.2. Write a Dockerfile

Create a Dockerfile:

```Dockerfile
# Use Node.js base image
FROM node:14

# Set working directory
WORKDIR /usr/src/app

# Copy application files
COPY package*.json ./
COPY app.js ./

# Install dependencies
RUN npm install

# Expose port
EXPOSE 3000

# Start the app
CMD ["npm", "start"]
```

### 1.3. Build the Docker Image

Build the Docker image:

```bash
docker build -t my-app:1.0 .
```

Verify the image:

```bash
docker images
```

### 1.4. Push the Image to a Registry

Tag the image for a container registry (e.g., Docker Hub or a private registry):

```bash
docker tag my-app:1.0 <your-dockerhub-username>/my-app:1.0
```

Log in to Docker Hub:

```bash
docker login
```

Push the image:

```bash
docker push <your-dockerhub-username>/my-app:1.0
```

## Step 2: Deploy the Application on Kubernetes

### 2.1. Create a Deployment Manifest

Create a Kubernetes manifest file deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: <your-dockerhub-username>/my-app:1.0
        ports:
        - containerPort: 3000
```

Apply the manifest:

```bash
kubectl apply -f deployment.yaml
```

## 2.2. Expose the Deployment

Expose the deployment as a service:

```bash
kubectl expose deployment my-app-deployment --type=NodePort --port=3000
```

## Step 3: Access the Application

Get the NodePort:

```bash
kubectl get svc
```

Access the app in your browser or via curl:

```bash
curl http://<node-ip>:<node-port>
```

## Step 4: Verify and Manage

### 4.1. Check the Deployment

```bash
kubectl get deployments
kubectl describe deployment my-app-deployment
```

### 4.2. Check the Pods

```bash
kubectl get pods
```

### 4.3. Scale the Deployment

Scale to 5 replicas:

```bash
kubectl scale deployment my-app-deployment --replicas=5
```

Verify the updated pods:

```bash
kubectl get pods
```

## Step 5: Cleanup

To remove the resources:

```bash
kubectl delete deployment my-app-deployment
kubectl delete service my-app-deployment
```

By following this process, you build a Docker image, push it to a registry, and deploy it on a Kubernetes cluster. You can adapt the application, Dockerfile, and Kubernetes manifests to suit your needs.
