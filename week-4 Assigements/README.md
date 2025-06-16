
# **Week-4 Assignment**

---

# **Introduction to containerization and Docker fundamentals, Basic Commands**

#### What is Containerization?

Containerization is a lightweight form of virtualization that allows you to package an application with all of its dependencies and run it in isolated environments called **containers**.

#### Key Benefits:

* **Portability** – Run anywhere (development, test, production).
* **Consistency** – Same environment across machines.
* **Isolation** – Apps are isolated from each other.
* **Lightweight** – Uses less resources than traditional VMs.

---

### What is Docker?

**Docker** is an open-source platform used to **build, ship, and run containers**. It allows developers to automate deployment of applications inside lightweight containers.

#### Docker Architecture

1. **Docker Engine** – Core service that runs Docker.
2. **Docker Image** – Blueprint for containers.
3. **Docker Container** – Running instance of an image.
4. **Dockerfile** – Script with instructions to build an image.
5. **Docker Hub** – Cloud-based registry to store and share Docker images.

---

###  Basic Docker Commands

#### 1. Install Docker

Refer to [Docker's official site](https://docs.docker.com/get-docker/) for your OS.

---

#### 2. Docker Version

```bash
docker --version
docker info
```

---

#### 3. Pull an Image

Downloads an image from Docker Hub.

```bash
docker pull ubuntu
```

---

####  4. Run a Container

Starts a new container from an image.

```bash
docker run ubuntu
```

With interactive terminal:

```bash
docker run -it ubuntu
```

Detached mode:

```bash
docker run -d ubuntu
```

With a name:

```bash
docker run --name mycontainer ubuntu
```

---

#### 5. List Containers

* Running containers:

  ```bash
  docker ps
  ```

* All containers (including stopped):

  ```bash
  docker ps -a
  ```

---

####  6. Stop / Start / Restart Container

```bash
docker stop <container_id_or_name>
docker start <container_id_or_name>
docker restart <container_id_or_name>
```

---

####  7. Remove Container / Image

```bash
docker rm <container_id_or_name>
docker rmi <image_id_or_name>
```

---

####  8. View Logs / Exec into Container

```bash
docker logs <container_id_or_name>
docker exec -it <container_id_or_name> bash
```

---

#### 9. Build Docker Image from Dockerfile

```bash
docker build -t myimage:tag .
```

---

####  10. Tag & Push Image to Docker Hub

```bash
docker tag myimage username/myimage:tag
docker push username/myimage:tag
```

>  Ensure you're logged in with:

```bash
docker login
```

---

------------------------------------------------------------------------------------------
# **Docker installation and basic container operations, Build an image from Dockerfile**
------------------------------------------------------------------------------------------

Here’s a complete guide covering:

1. **Installing Docker**
2. **Basic Docker container operations**
3. **Building a Docker image using a Dockerfile**

---

### 1. Docker Installation

#### For **Ubuntu/Linux**:

```bash
# Update packages
sudo apt-get update

# Install required packages
sudo apt-get install ca-certificates curl gnupg

# Add Docker’s official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Check Docker version
docker --version
```

#### For **Windows/Mac**:

* Download and install **Docker Desktop** from:
   [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
* After installation, check via terminal:

```bash
docker --version
```

---

### 2. Basic Docker Container Operations

#### Check Docker status:

```bash
sudo systemctl status docker
```

#### Pull an image:

```bash
docker pull ubuntu
```

#### Run a container:

```bash
docker run -it ubuntu
```

#### Run with custom name:

```bash
docker run --name myubuntu -it ubuntu
```

#### List running containers:

```bash
docker ps
```

#### List all containers:

```bash
docker ps -a
```

#### Stop a container:

```bash
docker stop <container_id or name>
```

#### Remove a container:

```bash
docker rm <container_id or name>
```

#### Remove an image:

```bash
docker rmi <image_id or name>
```

---

### 3. Build Docker Image from Dockerfile


#### Sample `Dockerfile`:

```Dockerfile
# Base image
FROM python:3.9

# Working directory
WORKDIR /app

# Copy files
COPY app.py .

# Install dependencies (if any)
# RUN pip install -r requirements.txt

# Command to run the app
CMD ["python", "app.py"]
```

#### Sample `app.py`:

```python
print("Hello from Docker container!")
```

#### Build image:

```bash
docker build -t my-python-app .
```

#### Run the container:

```bash
docker run my-python-app
```

---

---------------------------------------------------------------------------------------
# **Docker Registry, DockerHub, Create a Multi-Stage Build**
-------------------------------------------------------------------------------------



### 1. Docker Registry & DockerHub

#### **Docker Registry**

A **Docker Registry** is a **storage and distribution system for Docker images**. It can be:

* **Public** (like Docker Hub)
* **Private** (self-hosted or cloud)

 Docker uses **Docker Hub** as the default public registry.

---

#### **Docker Hub**

[Docker Hub](https://hub.docker.com) is a **cloud-based repository** where you can:

* Store and share container images
* Pull official and community-maintained images

 **To use Docker Hub:**

1. Create an account: [https://hub.docker.com/signup](https://hub.docker.com/signup)
2. Login via CLI:

   ```bash
   docker login
   ```

---

### 2. Push/Pull Docker Images to/from DockerHub

#### **Push an image**

Suppose you built an image:

```bash
docker build -t my-python-app .
```

To push it to Docker Hub:

1. **Tag the image** with your Docker Hub username:

   ```bash
   docker tag my-python-app yourdockerhubusername/my-python-app
   ```

2. **Push the image**:

   ```bash
   docker push yourdockerhubusername/my-python-app
   ```

 Now it’s available in your Docker Hub repository.

---

#### **Pull an image**

To pull it from Docker Hub on any system:

```bash
docker pull yourdockerhubusername/my-python-app
```

---

### 3. Multi-Stage Docker Build

#### Why Multi-Stage?

It helps to:

* Reduce image size
* Remove build-time dependencies
* Keep only runtime artifacts

---

#### Example: Go App Multi-Stage Dockerfile

##### Structure:

```
goapp/
├── Dockerfile
└── main.go
```

##### main.go:

```go
package main
import "fmt"
func main() {
    fmt.Println("Hello from Go in Docker!")
}
```

##### Dockerfile:

```Dockerfile
# Stage 1: Build
FROM golang:1.20 AS builder

WORKDIR /app
COPY main.go .
RUN go build -o app

# Stage 2: Runtime
FROM alpine:latest

WORKDIR /app
COPY --from=builder /app/app .

CMD ["./app"]
```

---

#### Build:

```bash
docker build -t go-multistage-app .
```

#### Run:

```bash
docker run go-multistage-app
```

 Output:

```
Hello from Go in Docker!
```

---



--------------------------------------------------------------------------------------
# **Push and pull image to Docker hub and ACR**
--------------------------------------------------------------------------------------


---

### Part 1: Push & Pull Docker Images to/from Docker Hub

#### Step 1: Login to Docker Hub

```bash
docker login
```

> Enter your Docker Hub username and password.

---

#### Step 2: Tag the Image

```bash
docker tag <local-image-name> <dockerhub-username>/<repository-name>:<tag>
```

**Example:**

```bash
docker tag myapp:latest chanduo5/myapp:latest
```

---

#### Step 3: Push the Image

```bash
docker push chanduo5/myapp:latest
```

---

#### Step 4: Pull the Image (from another machine or later)

```bash
docker pull chanduo5/myapp:latest
```

---

### Part 2: Push & Pull Docker Images to/from Azure Container Registry (ACR)

#### Step 1: Create an Azure Container Registry (ACR)

Using Azure CLI:

```bash
az acr create --resource-group <resource-group-name> \
  --name <registry-name> --sku Basic
```

**Example:**

```bash
az acr create --resource-group myResourceGroup --name myRegistry123 --sku Basic
```

---

#### Step 2: Login to ACR

```bash
az acr login --name <registry-name>
```

**Example:**

```bash
az acr login --name myRegistry123
```

---

#### Step 3: Get the ACR Login Server

```bash
az acr list --resource-group <resource-group-name> --query "[].{acrLoginServer:loginServer}" --output table
```

Result will be something like:

```
myregistry123.azurecr.io
```

---

#### Step 4: Tag the Image for ACR

```bash
docker tag <local-image-name> <acr-login-server>/<repository-name>:<tag>
```

**Example:**

```bash
docker tag myapp:latest myregistry123.azurecr.io/myapp:latest
```

---

#### Step 5: Push the Image to ACR

```bash
docker push myregistry123.azurecr.io/myapp:latest
```

---

#### Step 6: Pull the Image from ACR

```bash
docker pull myregistry123.azurecr.io/myapp:latest
```

---



------------------------------------------
# **Create a Custom Docker Bridge Network**
------------------------------------------


---

### What is a Custom Docker Bridge Network?

By default, Docker creates a `bridge` network for containers. However, creating a **custom bridge network** provides:

* Better control over container communication
* Automatic DNS-based name resolution between containers
* More isolation and security

---

### Step-by-Step: Create a Custom Bridge Network

#### Step 1: Create the Custom Network

```bash
docker network create \
  --driver bridge \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.1 \
  custom-bridge-net
```

You can omit `--subnet` and `--gateway` if you want Docker to assign them automatically:

```bash
docker network create custom-bridge-net
```

---

#### Step 2: Verify the Network

```bash
docker network ls
```

Look for `custom-bridge-net` in the list.

To inspect details:

```bash
docker network inspect custom-bridge-net
```

---

#### Step 3: Run Containers on the Custom Network

Run container A:

```bash
docker run -dit --name containerA --network custom-bridge-net alpine sh
```

Run container B:

```bash
docker run -dit --name containerB --network custom-bridge-net alpine sh
```

---

#### Step 4: Test Communication

Now, exec into one container and ping the other by name:

```bash
docker exec -it containerA sh
```

Inside the container:

```sh
ping containerB
```

If everything is correct, `containerA` will resolve `containerB` by name and ping it via the custom bridge network.

---

#### Step 5: Clean Up

```bash
docker container rm -f containerA containerB
docker network rm custom-bridge-net
```

---


--------------------------------------------------------------------
# **Create a Docker volume and mount it to a container.**
---------------------------------------------------------------------


---

### What is a Docker Volume?

A **volume** is a persistent storage mechanism in Docker, used to:

* Store data **outside** of the container filesystem.
* Share data between the host and containers or between multiple containers.
* Ensure data persistence even after the container is removed.

---

### Step-by-Step: Create and Mount a Docker Volume

#### Step 1: Create a Docker Volume

```bash
docker volume create myvolume
```

You can verify the volume:

```bash
docker volume ls
```

Inspect details:

```bash
docker volume inspect myvolume
```

---

#### Step 2: Run a Container with the Volume Mounted

Mount the volume to a specific directory inside the container:

```bash
docker run -dit \
  --name volume-container \
  -v myvolume:/app/data \
  ubuntu
```

> This mounts the Docker volume `myvolume` to `/app/data` inside the container.

---

#### Step 3: Use the Volume

Enter the container:

```bash
docker exec -it volume-container bash
```

Then inside the container:

```bash
cd /app/data
echo "Hello from volume" > hello.txt
```

Exit the container:

```bash
exit
```

---

#### Step 4: Verify Data Persistence

Stop and remove the container:

```bash
docker rm -f volume-container
```

Now run another container using the same volume:

```bash
docker run -it --rm -v myvolume:/app/data ubuntu cat /app/data/hello.txt
```

>  You’ll see the message:
> `Hello from volume`

This confirms that the data persisted even after the container was deleted.

---

---------------------------------------------------------------------------------------
# **Docker Compose for multi-container applications, Docker security best practices**
-----------------------------------------------------------------------------------------


### Part 1: Docker Compose for Multi-Container Applications

**Docker Compose** is a tool that allows you to define and run multi-container Docker applications using a single YAML file (`docker-compose.yml`).

---

#### Example: Web App + Redis (Python Flask + Redis)


---

#### `app.py`

```python
from flask import Flask
import redis

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = cache.incr('hits')
    return f'Hello! This page has been viewed {count} times.'

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

---

#### `requirements.txt`

```
flask
redis
```

---

#### `Dockerfile`

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

---

#### `docker-compose.yml`

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - redis
  redis:
    image: redis:alpine
```

---

#### Run the Application

```bash
docker-compose up --build
```

Access it at: `http://localhost:5000`

#### Stop & Remove

```bash
docker-compose down
```

---

### Part 2: Docker Security Best Practices

Security is critical in containerized environments. Below are key **Docker security best practices**:

---

#### 1. **Use Official and Minimal Base Images**

*  Prefer `python:3.9-slim` or `alpine` over full-blown images.
*  Pull from trusted sources.

---

#### 2. **Avoid Running as Root**

In your Dockerfile:

```Dockerfile
RUN adduser --disabled-password appuser
USER appuser
```

---

#### 3. **Use `.dockerignore` File**

Prevent sensitive or unnecessary files from being copied into the image.

Example:

```
__pycache__/
*.env
.git/
```

---

#### 4. **Limit Container Capabilities**

Run containers with least privileges:

```bash
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp
```

---

#### 5. **Use Read-Only Filesystems (Where Applicable)**

```bash
docker run --read-only ...
```

---

#### 6. **Scan Images for Vulnerabilities**

Use:

* `docker scan` (Docker Desktop)
* [Trivy](https://github.com/aquasecurity/trivy)
* Azure Defender for Containers (for cloud)

---

#### 7. **Keep Docker and Images Up-to-Date**

Update Docker engine regularly:

```bash
sudo apt update && sudo apt upgrade docker-ce
```

---

#### 8. **Use Private Registries**

Use services like:

* Docker Hub private repos
* Azure Container Registry (ACR)
* AWS ECR, GCR

---

#### 9. **Enable Docker Content Trust (DCT)**

Enforces image signing:

```bash
export DOCKER_CONTENT_TRUST=1
```

---
