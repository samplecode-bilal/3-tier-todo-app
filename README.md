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
docker build -t three-tier-frontend .
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
  --name backend backend:latest
```

- frontend:
```bash
docker run -d -p 3000:3000 \
  -e REACT_APP_BACKEND_URL=http://<public-ip>:8080/api/tasks \
  --name frontend frontend:latest
```
Replace <public-ip> with your EC2 public IP (e.g., http://54.123.45.67:8080/api/tasks).

