
# 🧩 **Docker Hands-On Lab (Part 2): Push to Docker Hub & Deploy on AWS EC2**

---

## 🎯 **Objectives**

By the end of this lab, you will:

* Push a custom Docker image to **Docker Hub**.
* Launch an **EC2 instance** on AWS.
* Install **Docker Engine** on EC2.
* Pull and run your published image from Docker Hub.
* Test your app running publicly on the EC2 instance.

---

## 🧱 **Pre-requisites**

✅ AWS account (Free Tier OK)
✅ SSH client (PuTTY / Terminal)
✅ Docker Hub account
✅ Image built locally (e.g., `getting-started`)

---

## ⚙️ **Task 1 – Push Image to Docker Hub**

1. **Login to Docker Hub**

   ```bash
   docker login
   ```

   Enter your Docker Hub username & password.

2. **Tag the image**

   ```bash
   docker tag getting-started varungupta2809/getting-started:v1
   ```

3. **Push the image**

   ```bash
   docker push varungupta2809/getting-started:v1
   ```

4. **Verify**
   Visit [https://hub.docker.com/repositories](https://hub.docker.com/repositories) →
   You should see `varungupta2809/getting-started:v1`.

---

## ☁️ **Task 2 – Launch an EC2 Instance**

1. **Login to AWS Console → EC2 Service → Launch Instance**

   * Name: `docker-app-server`
   * AMI: **Amazon Linux 2 (x86_64)**
   * Instance Type: **t2.micro**
   * Key pair: Create new or use existing.
   * Security Group Inbound Rules:

     | Type       | Port | Source               |
     | ---------- | ---- | -------------------- |
     | SSH        | 22   | Your IP              |
     | HTTP       | 80   | Anywhere (0.0.0.0/0) |
     | Custom TCP | 3000 | Anywhere (0.0.0.0/0) |

2. **Launch Instance** and note its **Public IP / DNS**.

---

## 🔑 **Task 3 – Connect to EC2 via SSH**

```bash
chmod 400 your-key.pem
ssh -i "your-key.pem" ec2-user@<public-ip>
```

---

## 🐳 **Task 4 – Install Docker on EC2**

```bash
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
```

Log out and log back in to apply group permissions.

✅ Check:

```bash
docker --version
docker ps
```

---

## 📦 **Task 5 – Pull & Run Your App from Docker Hub**

1. **Pull the image**

   ```bash
   docker pull varungupta2809/getting-started:v1
   ```

2. **Run the container**

   ```bash
   docker run -d -p 3000:3000 --name app varungupta2809/getting-started:v1
   ```

3. **Verify**

   ```bash
   docker ps
   ```

4. **Test in browser**

   ```
   http://<ec2-public-ip>:3000
   ```

   ✅ You should see your running Node.js todo app.

---

## 💾 **(Optional) Task 6 – Use Docker Compose on EC2**

1. **Install Docker Compose**

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   docker-compose version
   ```

2. **Create `docker-compose.yml`**

   ```bash
   vi docker-compose.yml
   ```

   Paste:

   ```yaml
   version: "3.7"
   services:
     app:
       image: varungupta2809/getting-started:v1
       ports:
         - "3000:3000"
       environment:
         MYSQL_HOST: mysql
         MYSQL_USER: root
         MYSQL_PASSWORD: secret
         MYSQL_DB: todos
       depends_on:
         - mysql

     mysql:
       image: mysql:8.0
       environment:
         MYSQL_ROOT_PASSWORD: secret
         MYSQL_DATABASE: todos
       volumes:
         - todo-mysql-data:/var/lib/mysql

   volumes:
     todo-mysql-data:
   ```

3. **Start the stack**

   ```bash
   docker-compose up -d
   docker ps
   ```

4. **Test again**

   ```
   http://<ec2-public-ip>:3000
   ```

---

## 🧹 **Task 7 – Cleanup**

```bash
docker-compose down
docker stop app
docker rm app
docker rmi varungupta2809/getting-started:v1
exit
```

---

## ✅ **Summary**

In this second part, you:

* Tagged and pushed a Docker image to Docker Hub
* Launched an EC2 instance and installed Docker
* Pulled and ran your image on the cloud
* Optionally deployed a multi-container stack using Compose

---
