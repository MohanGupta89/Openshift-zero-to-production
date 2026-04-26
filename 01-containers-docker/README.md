# 01 — Containers & Docker
## The Foundation — Understand This First

---

## 🤔 What is a Container? (Simple Explanation)

Think of shipping containers on a cargo ship.

Before shipping containers existed:
- Every ship had to be loaded differently
- Goods were packed in different ways
- Moving goods from ship to truck to train was a nightmare

After shipping containers:
- Everything fits in a standard box
- Works on any ship, truck, or train
- Load once, move anywhere

**Software containers work exactly the same way.**

Before containers:
```
"It works on my laptop but not on the server!"
- Every developer, every day
```

After containers:
```
"Works on my laptop = Works everywhere"
- Because the app + everything it needs is packed together
```

---

## 🐋 Docker = The Container Tool

Docker is the most popular tool to create and run containers.

```
Your App Code
    +
All Libraries it needs
    +
Configuration
    +
Operating System bits
    =
ONE DOCKER IMAGE (like a zip file)

Run that image anywhere = CONTAINER
```

---

## 🔧 Install Docker

```bash
# On Linux (RHEL/CentOS — your world brother!)
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker

# Check it works
docker version
docker run hello-world
```

---

## 📦 Docker Basics — Every Command Explained

### Images vs Containers
```
IMAGE = Recipe (blueprint, static, stored)
CONTAINER = Dish (running instance, active)

One image can create 100 containers
Like one recipe can make 100 pizzas
```

### Essential Docker Commands

```bash
# ===== IMAGES =====

# Download an image from internet
docker pull nginx

# See all images on your machine
docker images

# Delete an image
docker rmi nginx

# Search for images
docker search httpd


# ===== CONTAINERS =====

# Run a container (creates + starts)
docker run nginx

# Run in background (detached mode)
docker run -d nginx

# Run with a name
docker run -d --name my-web nginx

# Run and map port (host:container)
docker run -d -p 8080:80 --name my-web nginx
# Now visit: http://localhost:8080

# See running containers
docker ps

# See ALL containers (including stopped)
docker ps -a

# Stop a container
docker stop my-web

# Start a stopped container
docker start my-web

# Delete a container
docker rm my-web

# Delete running container forcefully
docker rm -f my-web


# ===== GOING INSIDE CONTAINER =====

# Open shell inside running container
docker exec -it my-web bash

# Run a command inside container
docker exec my-web ls /etc/nginx


# ===== LOGS =====

# See container logs
docker logs my-web

# Follow logs live (like tail -f)
docker logs -f my-web


# ===== CLEANUP =====

# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune

# Remove everything unused
docker system prune
```

---

## 📝 Dockerfile — Build Your Own Image

A Dockerfile is a recipe to build your image.

### Simple Example — Web App

```dockerfile
# Start FROM an existing image (base)
FROM nginx:latest

# Who made this image
LABEL maintainer="your-email@example.com"

# Copy your files into the image
COPY index.html /usr/share/nginx/html/

# Which port your app uses
EXPOSE 80

# Command to run when container starts
CMD ["nginx", "-g", "daemon off;"]
```

### Build and Run

```bash
# Build your image
docker build -t my-website:v1 .

# Run it
docker run -d -p 8080:80 my-website:v1

# Open browser: http://localhost:8080
```

---

## 🏗️ Real World Dockerfile — Node.js App

```dockerfile
# Use official Node image
FROM node:18-alpine

# Set working directory inside container
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy rest of app
COPY . .

# App runs on port 3000
EXPOSE 3000

# Start the app
CMD ["node", "server.js"]
```

---

## 💾 Docker Volumes — Persistent Storage

```bash
# Problem: Container data is lost when container dies
# Solution: Volumes

# Create a volume
docker volume create mydata

# Run container with volume mounted
docker run -d \
  --name my-db \
  -v mydata:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8

# Now even if container dies, data is safe in volume

# See all volumes
docker volume ls

# Delete a volume
docker volume rm mydata
```

---

## 🌐 Docker Networking

```bash
# Containers on same network can talk to each other

# Create a network
docker network create myapp-network

# Run database on that network
docker run -d \
  --name mysql-db \
  --network myapp-network \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8

# Run app on same network
docker run -d \
  --name my-app \
  --network myapp-network \
  -p 8080:80 \
  my-website:v1

# Now my-app can reach mysql-db by name: mysql-db:3306
```

---

## 🔄 Docker Compose — Multiple Containers Together

```yaml
# docker-compose.yml

version: '3'
services:
  
  # Web Application
  webapp:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - database
    networks:
      - myapp

  # Database
  database:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - myapp

volumes:
  dbdata:

networks:
  myapp:
```

```bash
# Start everything
docker-compose up -d

# Stop everything
docker-compose down

# See logs
docker-compose logs -f

# See status
docker-compose ps
```

---

## 🔐 Important Docker Concepts for OpenShift

### Docker Registry
```
Public Registry  = Docker Hub (hub.docker.com)
                   Anyone can pull images

Private Registry = Your own registry
                   Only your team can access
                   OpenShift has built-in private registry!

Image Name Format:
registry/username/imagename:tag

Examples:
nginx:latest                    ← Docker Hub official
docker.io/library/nginx:1.21   ← Full name
quay.io/openshift/origin:4.12  ← Red Hat Quay registry
```

### Image Tags
```bash
# Tag = version label
docker tag my-app:v1 my-app:latest
docker tag my-app:v1 myregistry.com/myteam/my-app:v1

# Push to registry
docker push myregistry.com/myteam/my-app:v1

# Pull from registry
docker pull myregistry.com/myteam/my-app:v1
```

---

## 🧪 Lab 01 — Build and Run Your First Container

```bash
# Step 1: Create a simple HTML file
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<body>
  <h1>My First Container!</h1>
  <p>Built by: Your Name</p>
</body>
</html>
EOF

# Step 2: Create Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF

# Step 3: Build image
docker build -t my-first-container:v1 .

# Step 4: Run it
docker run -d -p 8080:80 --name test-web my-first-container:v1

# Step 5: Test it
curl http://localhost:8080

# Step 6: See the logs
docker logs test-web

# Step 7: Go inside
docker exec -it test-web sh

# Step 8: Cleanup
docker stop test-web
docker rm test-web
docker rmi my-first-container:v1
```

---

## ❓ Interview Questions — Containers & Docker

**Q1: What is the difference between a container and a virtual machine?**
> VM = Full OS + App (heavy, slow to start, GBs in size)
> Container = Shared OS kernel + App only (lightweight, starts in seconds, MBs in size)
> Containers share the host OS kernel — that's why they're faster and smaller.

**Q2: What is the difference between Docker image and Docker container?**
> Image = static blueprint stored on disk (like a stopped machine)
> Container = running instance of an image (like a running machine)
> One image → many containers

**Q3: What is a Dockerfile?**
> A text file with instructions to build a Docker image. Each instruction creates a layer. Layers are cached so rebuilds are fast.

**Q4: What is Docker Hub?**
> Public registry where Docker images are stored and shared. Like GitHub but for container images.

**Q5: What happens to container data when it stops?**
> Data inside container filesystem is lost when container is deleted. Use Volumes to persist data outside the container lifecycle.

**Q6: How do containers communicate with each other?**
> Containers on the same Docker network can talk by container name. By default, containers are isolated. You must explicitly put them on the same network.

**Q7: What is a multi-stage Docker build?**
> Using multiple FROM statements in one Dockerfile. First stage builds/compiles. Second stage takes only the binary. Result = much smaller final image.

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Final stage — much smaller!
FROM alpine:latest
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

---

## ➡️ Next Step

Go to [02 — Kubernetes Basics](../02-kubernetes-basics/README.md)
