
# 🧩 **Docker / Kubernetes Hands-On Lab (Part 4): Deploy on AWS EKS**

---

## 🎯 **Objectives**

By the end of this lab you’ll:

* Create and configure an **EKS cluster** using `eksctl`
* Deploy your **Node.js app** + **MySQL** to EKS
* Expose your app publicly via a **LoadBalancer Service**
* Verify and test access using a browser or `curl`

---

## ⚙️ **Prerequisites**

✅ AWS account (with IAM admin or EKS permissions)
✅ AWS CLI v2 installed and configured (`aws configure`)
✅ `kubectl` installed
✅ `eksctl` installed → [https://eksctl.io](https://eksctl.io)
✅ Docker Hub image: `varungupta2809/getting-started:v1`

---

## ☁️ **Task 1 – Create EKS Cluster**

### 1️⃣ Create cluster + nodes

```bash
eksctl create cluster \
  --name todo-cluster \
  --region ap-south-1 \
  --nodegroup-name linux-nodes \
  --node-type t3.medium \
  --nodes 2
```

☕ Takes ~10–15 min.
✅ Check cluster status

```bash
kubectl get nodes
```

---

## 🗄️ **Task 2 – Create Namespace**

```bash
kubectl create namespace todo-app
kubectl config set-context --current --namespace=todo-app
```

---

## ⚙️ **Task 3 – Deploy MySQL**

### `mysql.yaml`

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
---
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

```bash
kubectl apply -f mysql.yaml
kubectl get pods,svc
```

---

## 🧩 **Task 4 – Deploy Node.js App**

### `app.yaml`

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
---
apiVersion: v1
kind: Service
metadata:
  name: todo-service
spec:
  type: LoadBalancer
  selector:
    app: todo-app
  ports:
    - port: 80
      targetPort: 3000
```

```bash
kubectl apply -f app.yaml
```

Check status:

```bash
kubectl get pods
kubectl get svc
```

---

## 🌐 **Task 5 – Access Your App**

Once the service status changes to `EXTERNAL-IP`:

```bash
kubectl get svc todo-service
```

Output example:

```
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP        PORT(S)        AGE
todo-service   LoadBalancer   10.0.187.12    a12b3cd4e5f.us-east-1.elb.amazonaws.com   80:31345/TCP   2m
```

Open in browser:

```
http://a12b3cd4e5f.us-east-1.elb.amazonaws.com
```

✅ You should see your Todo App running from EKS.

---

## 🧠 **Task 6 – Verify & Troubleshoot**

```bash
kubectl logs deployment/todo-app
kubectl exec -it deployment/mysql -- mysql -u root -p
# password: secret
show databases;
```

---

## 🧹 **Task 7 – Cleanup**

```bash
eksctl delete cluster --name todo-cluster --region ap-south-1
```

---

## ✅ **Summary**

You have now deployed a complete containerized app to **AWS EKS**, including:

* EKS cluster setup via `eksctl`
* Kubernetes Deployments + Services
* Internal Pod networking (MySQL ↔ App)
* External LoadBalancer access
* Clean cluster teardown

---

