
## Dockerization
1. **Clone the repository**
 git clone https://github.com/nyrahul/wisecow  

2. **Write Dockerfile**:
    ```Dockerfile
   # Use an official Node.js runtime as a parent image
FROM node:14

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install the application dependencies
RUN npm install

# Copy the application source code
COPY . .

# Expose the port the app runs on
EXPOSE 4499

# Command to run the application
CMD ["node", "wisecow.js"]
3. **Build the Docker image**:
    ```bash
    docker build -t wisecow-app .
    ```
4. **Test the Docker image locally**:
    ```bash
    docker run -p 8080:80 wisecow-app
    ```

## Kubernetes Deployment

1. **Create Deployment YAML (`wisecow-deployment.yaml`)**:
    ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
  name: wisecow
  spec:
   replicas: 1
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: <your-docker-image>
        ports:
        - containerPort: 4499
    ```
2. **Create Service YAML (`wisecow-service.yaml`)**:
    ```yaml
    apiVersion: v1
     kind: Service
    metadata:
  name: wisecow-service
  spec:
   type: LoadBalancer
  ports:
    - port: 80
      targetPort: 4499
  selector:
    app: wisecow
    ```
3. **Apply the manifest files**:
    ```bash
    kubectl apply -f wisecow-deployment.yaml
    kubectl apply -f wisecow-service.yaml
    ```

## Continuous Integration and Deployment (CI/CD)

1. **GitHub Actions Workflow**:
   - Create `.github/workflows/ci-cd.yml`:
     ```yaml
     name: CI/CD Pipeline

     on:
       push:
         branches:
           - main

     jobs:
       build-and-deploy:
         runs-on: ubuntu-latest
         steps:
         - name: Checkout code
           uses: actions/checkout@v2
         - name: Login to Container Registry
           uses: docker/login-action@v1
           with:
             username: ${{ secrets.REGISTRY_USERNAME }}
             password: ${{ secrets.REGISTRY_PASSWORD }}
         - name: Build Docker image
           run: docker build -t <your-container-registry>/wisecow-app:latest .
         - name: Push Docker image
           run: docker push <your-container-registry>/wisecow-app:latest
         - name: Deploy to Kubernetes
           run: kubectl apply -f wisecow-deployment.yaml
           env:
             KUBECONFIG: ${{ secrets.KUBE_CONFIG_DATA }}
     ```
   - Configure Docker and Kubernetes secrets in the GitHub repository settings.

## TLS Implementation

1. **Generate TLS Certificates**:
   - Use tools like Let's Encrypt to generate certificates.
   - kubectl create secret tls wisecow-tls --cert=path/to/tls.crt --key=path/to/tls.key
2. **Objective 1: System Health Monitoring Script (Python)**:
   - ```yaml
       import psutil
       import logging

# Set up logging
```yaml
          logging.basicConfig(filename='system_health.log', level=logging.INFO)

# Define thresholds
CPU_THRESHOLD = 80
MEMORY_THRESHOLD = 80
DISK_THRESHOLD = 80

# Check CPU usage
cpu_usage = psutil.cpu_percent()
if cpu_usage > CPU_THRESHOLD:
    logging.warning(f'CPU usage is high: {cpu_usage}%')

# Check memory usage
memory = psutil.virtual_memory()
if memory.percent > MEMORY_THRESHOLD:
    logging.warning(f'Memory usage is high: {memory.percent}%')

# Check disk usage
disk = psutil.disk_usage('/')
if disk.percent > DISK_THRESHOLD:
    logging.warning(f'Disk usage is high: {disk.percent}%')

print("Health check completed. Check 'system_health.log' for alerts.")
```
3.  **Application Health Checker (Python)**:
```yaml
import requests

def check_application(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            print("Application is UP")
        else:
            print(f"Application is DOWN, status code: {response.status_code}")
    except requests.exceptions.RequestException as e:
        print("Application is DOWN, error:", e)

# Replace with your application's URL
check_application('http://localhost :4499')


```

