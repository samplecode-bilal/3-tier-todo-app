## 3-Tier To-Do App Deployment (ReactJS + NodeJS + MongoDB) with GitHub, Docker Hub, and Kubernetes

This is a 3-tier to-do list application built with a React frontend, Node.js/Express backend, and MongoDB database. Users can create, view, update, and delete tasks, which are stored in MongoDB and displayed dynamically on the frontend, deployed using Docker and Kubernetes.

## Set Up Minikube:

```bash
# Install Docker
sudo su
sudo apt update && sudo apt -y install docker.io
sudo systemctl enable --now docker

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Install Minikube
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.24.0/minikube-linux-amd64
chmod +x minikube && sudo mv minikube /usr/local/bin/

# Start Minikube
sudo apt install conntrack -y
minikube start --vm-driver=none
minikube status
```

## 📜Code
## 🐳Build
```bash
git clone https://github.com/<your-username>/3-tier-todo-app.git
cd 3-tier-todo-app
```

- Create Docker Netowork
```bash
docker network create todo-network
```
- cd frontend/
```bash
docker build -t three-tier-frontend .
cd ..
```
- cd backend/
```bash
docker build -t three-tier-backend .
cd ..
```

### Run:
- mongo:
```bash
docker run -d -p 27017:27017 --network todo-network --name mongo mongo:latest
```
- backend:
```bash
docker run -d --network todo-network -p 8080:8080 \
  -e MONGO_CONN_STR=mongodb://mongo:27017/tododb \
  -e USE_DB_AUTH=false \
  --name backend three-tier-backend:latest
```

- frontend:
```bash
docker run -d -p 3000:3000 \
  -e REACT_APP_BACKEND_URL=http://<public-ip>:8080/api/tasks \
  --name frontend three-tier-frontend:latest
```
Replace <public-ip> with your EC2 public IP (e.g., http://54.123.45.67:8080/api/tasks).
Use the following command sequence to access the MongoDB container’s shell interface:
```bash
docker exec -it mongo bash 
show dbs;

use tododb 
db.tasks.find()

exit 
exit 
```

### Push
```bash
docker login
docker tag three-tier-frontend YOUR-DOCKERHUB-USERNAME/three-tier-frontend:latest
docker push YOUR-DOCKERHUB-USERNAME/three-tier-frontend:latest


docker tag three-tier-backend YOUR-DOCKERHUB-USERNAME/three-tier-backend:latest
docker push YOUR-DOCKERHUB-USERNAME/three-tier-backend:latest

```

## ☸️ Deploy
Note: please create namespace and change imagename in deployment.yaml file of frontend and backend.
```bash
kubectl create namespace workshop
cd k8s
kubectl apply -f secrets.yaml -n workshop
kubectl apply -f mongodb-deployment.yaml -n workshop
kubectl apply -f mongodb-service.yaml -n workshop
kubectl apply -f backend-deployment.yaml -n workshop
kubectl apply -f backend-service.yaml -n workshop
kubectl apply -f frontend-deployment.yaml -n workshop
kubectl apply -f frontend-service.yaml -n workshop
```
If you want to go to inside mongodb pod:
```bash
kubectl get pods -n workspace
kubectl exec -it mongodb-c5b44f89c-gkccv -- bash
mongosh
admin
# Enter password: password123

show dbs
use tododb 
db.tasks.find()

exit 
exit 
```

After lab, please delete resources. 
