
# 🧩 **Docker Hands-On Lab (Part 3): Deploy Containers on Kubernetes using Minikube (Simplified)**

---

## 🎯 **Objective**

By the end of this lab, you’ll:

* Install **Minikube** locally (using Docker as the driver)
* Deploy your **Node.js app** and **MySQL** containers in Kubernetes
* Expose the app using a **Service**
* Access the app via browser on `localhost`

---

## ⚙️ **Prerequisites**

✅ Docker Desktop or Docker Engine installed
✅ kubectl installed (`kubectl version --client`)
✅ Docker Hub account (image: `varungupta2809/getting-started:v1`)

---

## 🧱 **Task 1 – Install & Start Minikube**

### 🔹 Windows (PowerShell)

```bash
choco install minikube -y
```

### 🔹 macOS (Homebrew)

```bash
brew install minikube
```

### 🔹 Linux

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### 🔹 Start Minikube (using Docker)

```bash
minikube start --driver=docker
```

✅ Verify

```bash
kubectl get nodes
minikube status
```

---

## 📦 **Task 2 – Create Kubernetes Manifest Files**

Create a folder:

```bash
mkdir k8s-todo-app && cd k8s-todo-app
```

---

### 🐬 1️⃣ `mysql-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: secret
            - name: MYSQL_DATABASE
              value: todos
          ports:
            - containerPort: 3306
```

---

### 🌐 2️⃣ `mysql-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
  clusterIP: None
```

👉 Using `clusterIP: None` enables direct Pod-to-Pod networking (simplifies DNS resolution).

---

### 🧩 3️⃣ `app-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo-app
  template:
    metadata:
      labels:
        app: todo-app
    spec:
      containers:
        - name: todo-app
          image: varungupta2809/getting-started:v1
          env:
            - name: MYSQL_HOST
              value: mysql
            - name: MYSQL_USER
              value: root
            - name: MYSQL_PASSWORD
              value: secret
            - name: MYSQL_DB
              value: todos
          ports:
            - containerPort: 3000
```

---

### 🚀 4️⃣ `app-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: todo-service
spec:
  type: NodePort
  selector:
    app: todo-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30080
```

---

## ⚙️ **Task 3 – Deploy to Kubernetes**

Apply all manifests:

```bash
kubectl apply -f .
```

Check status:

```bash
kubectl get pods
kubectl get svc
```

✅ You should see:

```
NAME            TYPE       CLUSTER-IP     PORT(S)          AGE
mysql           ClusterIP  None           3306/TCP         1m
todo-service    NodePort   10.x.x.x       3000:30080/TCP   1m
```

---

## 🌐 **Task 4 – Access the Application**

Start Minikube tunnel (optional for Windows):

```bash
minikube tunnel
```

Or get direct URL:

```bash
minikube service todo-service --url
```

Example:

```
http://127.0.0.1:30080
```

✅ Open in browser → You’ll see your Todo App running and connected to MySQL.

---

## 🧠 **Task 5 – Verify Pods & Logs**

```bash
kubectl get all
kubectl logs deployment/todo-app
kubectl exec -it deployment/mysql -- mysql -u root -p
```

Enter password: `secret`

```sql
show databases;
```

---

## 🧹 **Task 6 – Cleanup**

```bash
kubectl delete -f .
minikube stop
minikube delete
```

---

## ✅ **Summary**

In this simplified lab, you:

* Installed **Minikube using Docker**
* Created **Deployments** and **Services** for MySQL and Node.js
* Verified **Kubernetes networking (Pod ↔ Pod)**
* Exposed your app via **NodePort**
* Accessed it locally via **Minikube service**

---

