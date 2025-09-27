# 🚀 Docker Complete App

This is a simple full-stack demo application designed to demonstrate how to use **Docker** and **Docker Compose** to containerize and run a backend API, a frontend client, and a MongoDB database.

### 🛠️ Installation
Before running this demo, make sure you have Docker installed on your system.

## 📥 Install Docker
- [Docker Desktop for Mac & Windows](https://www.docker.com/products/docker-desktop)
- [Docker Engine for Linux](https://docs.docker.com/engine/install/)
After installation, verify it's working:
```bash
docker --version
docker compose version
```

---

## 🧱 Stack

- **Frontend**: React (or any frontend framework)
- **Backend**: Node.js with Express
- **Database**: MongoDB
- **Containerization**: Docker + Docker Compose

---

## 📦 Features

- Fully containerized with Docker
- Docker Compose orchestration for multi-service setup
- Live reload for development via volume mounting
- Environment variables handled via `.env` files

---

## 🚀 Getting Started

### 1. Clone the Repo

```bash
git clone https://github.com/joserojasv/docker-complete.git
cd docker-complete
```

## 🐳 Check Existing Docker Images and Containers

🔍 View local Docker images:
```bash
docker images
```

🔍 View all containers (running + stopped):
```bash
docker ps -a
```

## ⚙️ Running the Application
This app uses multiple containers that need to communicate with each other. 
</br>We have two ways to run it:</br>
**Imperative:** Run the docker build, docker run, and other commands manually in the terminal.</br>
**Declarative:** Use docker-compose to configure and start everything with a single command.

### --> Imperative Method (Manual)
1. Create a private Docker network:
```bash
docker network create goals-net
docker network ls
```
2. **MongoDB** (no build needed, using official image):
```bash
docker run --name mongodb --rm -v data:/data/db -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=secret mongo
```
3. **Backend**:
Build the backend image:
```bash
docker build -t goals-node ./backend
```
Run the backend container:
```bash
docker run --name goals-backend -v /Users/jose.rojas/develop/docker-complete/backend:/app -v logs:/app/logs -v /app/node_modules -d --rm  -p 80:80 --network goals-net goals-node
```
4. **Frontend**:
Build the frontend image:
```bash
docker build -t goals-react ./frontend
```

Run the frontend container:
```bash
docker run -v /Users/jose.rojas/develop/docker-complete/frontend/src:/app/src --name goals-frontend --network goals-net --rm -p 3000:3000 -it goals-react
```

### --> Declarative Method (Using Docker Compose)
Instead of running multiple commands, use **docker-compose** to build and start all containers together.
To start the app with Docker Compose:

```bash
docker compose up -d --build
```

```bash
# -> Example for Docker Compose File <-

services:
  mongodb:
    image: mongo
    volumes:
      - data:/data/db
    env_file:
      - ./env/mongo.env
    # Docker Compose creates a default network automatically

  backend:
    build: ./backend
    image: jrojascr/docker-complete-backend:latest
    ports:
      - "80:80"
    volumes:
      - logs:/apps/logs           # Named volume (declared below)
      - ./backend:/app            # Bind mount for live code reloads
      - /app/node_modules         # Anonymous volume to preserve container's node_modules
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb

  frontend:
    build: ./frontend
    image: jrojascr/docker-complete-frontend:latest
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src   # Bind mount to sync source files
    stdin_open: true              # Keeps STDIN open (equivalent to `-i`)
    tty: true                     # Allocates TTY (equivalent to `-t`)
    depends_on:
      - backend

volumes:
  data:
  logs:
```
### More on Docker Compose. 
Make sure your .env files (./env/mongo.env, ./env/backend.env) are properly set up with environment variables.
From the root of your project (where the docker-compose.yml is located), run:
```bash
ℹ️ INFO
docker compose up --build # Start and build all containers together.
docker compose up -d # Start all containers together in detached mode (in the background)

⚠️ WARNING
docker compose stop # Just stops containers (does not remove them)
docker compose down # Stops and removes containers + networks
docker compose down -v # Stops and removes containers, networks, and volumes
```
This will:</br>

- ✅ **Build** the backend and frontend images from their respective directories.
- 🐳 **Start** the MongoDB, backend, and frontend containers using Docker Compose.
- 📦 **Create** the named volumes (`data` and `logs`) automatically.
- 🌐 **Set up** the default Docker network so containers can communicate with each other.


### 🗺️ Architecture Overview

```text
+--------------------------+
| Frontend Container       |
| Service: frontend        |
| Image: jrojascr/docker-complete-frontend |
| Port: 3000               |
+--------------------------+
            |
            |  HTTP requests (e.g. user submits form)
            v
+--------------------------+
| Backend Container        |
| Service: backend         |
| Image: jrojascr/docker-complete-backend  |
| Port: 80                 |
+--------------------------+
            |
            |  TCP connection (using credentials)
            v
+--------------------------+
| MongoDB Container        |
| Service: mongodb         |
| Image: mongo             |
| Port: 27017 (default)    |
+--------------------------+
```
- All containers run in the same Docker network (via Docker Compose)
- Frontend communicates with Backend via HTTP (e.g. http://backend:80)
- Backend connects to MongoDB using credentials and service name
- Each container is a service defined in `docker-compose.yaml`

## 📤 Pushing Images to Docker Hub

Once you've built your Docker images locally, you can share them with others by uploading them to [Docker Hub](https://hub.docker.com).

#### 🪪 1. Log in to Docker Hub
```bash
docker login
```
🏷️ 2. Tag your images
You must tag your image using the format:
<your-dockerhub-username>/<repository-name>:<tag>

```bash
# Example for backend
docker tag goals-node jrojascr/docker-complete-backend:latest

# Example for frontend
docker tag goals-react jrojascr/docker-complete-frontend:latest
```

🚀 3. Push the images to Docker Hub
```bash
# Push backend
docker push jrojascr/docker-complete-backend:latest

# Push frontend
docker push jrojascr/docker-complete-frontend:latest
```
🔐 Make sure the repository name on Docker Hub matches what you're pushing. You can create the repo manually at: https://hub.docker.com/repositories

🧪 4. Verify from another computer
To pull the image on a different machine:

```bash
docker pull jrojascr/docker-complete-backend:latest
docker pull jrojascr/docker-complete-frontend:latest
```

## ✏️ Docker Useful Commands


See the logs
docker logs <container_name_or_id>

⚠️ WARNING

**Be careful when running these commands! They can delete containers, images, and more.**</br>

1. Stop and remove all containers
```bash
docker stop $(docker ps -q) && docker rm $(docker ps -aq)
```

3. Remove single container
```bash
docker rm <container_name_or_id>
```

4. Remove all images 
```bash
docker rmi $(docker images -q)
```

5. Remove single image
```bash
docker rmi <container_name_or_id>
```
6. Remove volume
```bash
docker volume rm <volume_name_or_id>
```

🔥 Optional full cleanup:
If you're just cleaning up everything (containers, images, networks, build cache):
```bash
docker system prune -a
```
⚠️ This will delete:
All **stopped** containers
All images not used by running containers
All networks not used
All build cache
You’ll be prompted to confirm before it runs.