# 🧩 **Docker Hands-On Lab: From Containerization to Compose**

---

## 🧱 **Task 1 – Containerize the App**

### Steps

1. **Create a folder**

   ```bash
   mkdir docker-lab && cd docker-lab
   ```

2. **Clone the sample repo**

   ```bash
   git clone https://github.com/docker/getting-started.git
   cd getting-started/app
   code .
   ```

3. **Create a `Dockerfile`**

   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   CMD ["node", "src/index.js"]
   EXPOSE 3000
   ```

4. **Build the image**

   ```bash
   docker build -t getting-started .
   docker image ls
   ```

5. **Run a container**

   ```bash
   docker run -d -p 3000:3000 --name app getting-started
   docker ps
   ```

6. **Test in browser**
   👉 [http://localhost:3000](http://localhost:3000)

---

## ✏️ **Task 2 – Modify the Code**

1. Edit
   `src/static/js/app.js` → modify **line 56**.

2. Stop and remove old container

   ```bash
   docker stop app
   docker rm app
   ```

3. Rebuild and rerun

   ```bash
   docker build -t getting-started .
   docker run -d -p 3000:3000 --name app getting-started
   ```

4. Test again
   👉 [http://localhost:3000](http://localhost:3000)

---

## 🌍 **Task 3 – Share the App on Docker Hub**

1. **Tag image**

   ```bash
   docker tag getting-started varungupta2809/getting-started
   ```

2. **Login and push**

   ```bash
   docker login
   docker push varungupta2809/getting-started
   ```

3. **Run from Hub**

   ```bash
   docker run -d -p 3000:3000 --name app varungupta2809/getting-started
   ```

---

## 💾 **Task 4 – Persist the Database**

1. Run the container (no volume)

   ```bash
   docker run -d -p 3000:3000 --name app varungupta2809/getting-started
   ```

   ➤ Add some items → stop & remove → re-run → data lost.

2. Create a named volume

   ```bash
   docker volume create todo-db
   ```

3. Run with mounted volume

   ```bash
   docker run -d -p 3000:3000 --name app \
     --mount type=volume,src=todo-db,target=/etc/todos \
     getting-started
   ```

✅ Data now persists across container restarts.

---

## 🗄️ **Task 5 – MySQL Database Container**

1. **Pull image**

   ```bash
   docker pull mysql:8.0
   docker image inspect mysql:8.0
   ```

2. **Run container**

   ```bash
   docker run -d --name mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:8.0
   ```

3. **Connect inside**

   ```bash
   docker exec -it mysql mysql -u root -p
   # password: secret
   show databases;
   show tables;
   exit;
   ```

---

## 🔗 **Task 6 – Multi-Container App**

1. Stop and remove old containers

   ```bash
   docker stop mysql app && docker rm mysql app
   ```

2. Create network

   ```bash
   docker network create todo-app
   ```

3. Run MySQL in the network

   ```bash
   docker run -d --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:8.0
   ```

4. Run App connected to MySQL

   ```bash
   docker run -d -p 3000:3000 --name app \
     -w /app --network todo-app \
     -e MYSQL_HOST=mysql \
     -e MYSQL_USER=root \
     -e MYSQL_PASSWORD=secret \
     -e MYSQL_DB=todos \
     varungupta2809/getting-started
   ```

5. Test → [http://localhost:3000](http://localhost:3000)

6. Verify data in MySQL

   ```bash
   docker exec -it <mysql_container_id> mysql -u root -p
   ```

---

## ⚙️ **Task 7 – Orchestrate with Docker Compose**

1. **Create `docker-compose.yml`**

   ```yaml
   version: "3.7"
   services:
     app:
       image: getting-started:latest
       container_name: app
       working_dir: /app
       ports:
         - 3000:3000
       environment:
         MYSQL_HOST: mysql
         MYSQL_USER: root
         MYSQL_PASSWORD: secret
         MYSQL_DB: todos
       depends_on:
         - mysql

     mysql:
       image: mysql:8.0
       container_name: mysql
       volumes:
         - todo-mysql-data:/var/lib/mysql
       environment:
         MYSQL_ROOT_PASSWORD: secret
         MYSQL_DATABASE: todos

   volumes:
     todo-mysql-data:
   ```

2. **Start all**

   ```bash
   docker-compose up -d
   docker ps
   ```

3. **Test**
   👉 [http://localhost:3000](http://localhost:3000)

4. **Tear down**

   ```bash
   docker-compose down
   ```

---

### ✅ **Outcome**

By completing this lab, you have:

* Built, run, and modified a Node.js container
* Published an image to Docker Hub
* Used named volumes for persistence
* Connected app and MySQL containers via custom network
* Deployed multi-container architecture with Docker Compose

---

