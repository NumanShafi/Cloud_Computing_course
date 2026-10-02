# Docker Hands-On Lab: From Installation to Building and Running Applications

## Course/Lab Goal

By the end of this lab, students should be comfortable with:

- Docker, images, containers, registries, and Docker Engine
- Installing and verifying Docker
- Running containers from public images
- Managing the container lifecycle
- Ports and container networking
- Environment variables
- Images and Dockerfiles
- Building custom images
- Running a Flask application inside Docker
- Volumes and persistent data
- Docker networks
- Docker Compose
- Logs, inspection, and troubleshooting
- Docker Hub
- Cleanup and basic security

> **Recommended environment:** Ubuntu Linux. Commands are primarily for Ubuntu/Debian.

---

# 1. Docker in Simple Words

Docker packages an application together with the environment it needs so it can run consistently on different machines.

### Simple mental model

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | docker run
    v
Container
    |
    v
Running Application
```

### Important terms

| Term | Simple meaning |
|---|---|
| Docker Engine | Software that runs containers |
| Image | Read-only template used to create containers |
| Container | Running/isolated instance of an image |
| Dockerfile | Instructions for building an image |
| Registry | Server that stores images |
| Docker Hub | Public container image registry |
| Volume | Persistent storage managed by Docker |
| Network | Communication mechanism between containers |
| Docker Compose | Tool for defining multi-container applications |

---

# 2. Docker Architecture

```text
+-----------------------------+
|        Docker Client        |
|       docker command        |
+-------------+---------------+
              |
              | Docker API
              v
+-----------------------------+
|        Docker Engine        |
|                             |
|  Images                     |
|  Containers                 |
|  Networks                   |
|  Volumes                    |
+-------------+---------------+
              |
              v
       Running Containers
```

When you execute:

```bash
docker run nginx
```

Docker approximately:

1. Checks whether the `nginx` image exists locally.
2. Downloads it if necessary.
3. Creates a container.
4. Starts the container.
5. Runs the image's configured command.

---

# 3. Install Docker on Ubuntu

## Step 1: Update packages

```bash
sudo apt update
```


## Step 2: Remove conflicting old packages

```bash
sudo apt remove -y docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc
```


## Step 3 to 7: Install Docker


```bash
curl -fsSL https://get.docker.com -o get-docker.sh  

DRY_RUN=1 sudo sh ./get-docker.sh
```

<img width="1650" height="799" alt="image" src="https://github.com/user-attachments/assets/a9f109b0-faa2-490f-8de8-d5fc1799ae2e" />


## Step 8: Check Docker service

```bash
sudo systemctl status docker
```

If necessary:

```bash
sudo systemctl enable --now docker
```

## Step 9: Check version

```bash
docker --version
```

```bash
docker version
```

## Step 10: Test Docker

```bash
sudo docker run hello-world
```

If the Docker welcome message appears, Docker is working.

---

# 4. Run Docker Without sudo

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Apply the group membership:

```bash
newgrp docker
```

Test:

```bash
docker run hello-world
```

Check:

```bash
docker ps
```

> **Security note:** Membership in the `docker` group effectively gives root-level control over the host. Use it only for trusted users.

If it still requires `sudo`, log out and log in again.

---

# 5. First Container: Hello World

Run:

```bash
docker run hello-world
```

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

The `hello-world` container normally exits after printing its message.

### Hands-on Task 1

Students should:

1. Run `hello-world`.
2. Run `docker ps`.
3. Run `docker ps -a`.
4. Explain why the container is stopped.
5. Find the container ID.

---

# 6. Docker Images

List local images:

```bash
docker images
```

Modern equivalent:

```bash
docker image ls
```

Pull an image:

```bash
docker pull nginx
```

Pull a specific tag:

```bash
docker pull nginx:alpine
```

Inspect:

```bash
docker image inspect nginx
```

View image history:

```bash
docker image history nginx
```

Remove:

```bash
docker image rm nginx:alpine
```

---

# 7. Image Tags

Images commonly use:

```text
repository:tag
```

Examples:

```text
nginx:latest
nginx:alpine
python:3.12
python:3.12-slim
ubuntu:24.04
```

If you run:

```bash
docker pull nginx
```

Docker normally uses the `latest` tag.

For reproducible applications, use intentional versions rather than blindly depending on `latest`.

---

# 8. Run an Nginx Container

Run:

```bash
docker run nginx
```

This runs in the foreground.

Stop with:

```text
Ctrl + C
```

Run detached:

```bash
docker run -d nginx
```

Check:

```bash
docker ps
```

---

# 9. Container Names

Run with a custom name:

```bash
docker run -d --name my-nginx nginx
```

Check:

```bash
docker ps
```

Stop:

```bash
docker stop my-nginx
```

Start:

```bash
docker start my-nginx
```

Restart:

```bash
docker restart my-nginx
```

Remove:

```bash
docker rm my-nginx
```

---

# 10. Container Lifecycle

Important commands:

```bash
docker create
docker start
docker run
docker stop
docker restart
docker kill
docker pause
docker unpause
docker rm
```

### Create without starting

```bash
docker create --name test-nginx nginx
```

Start:

```bash
docker start test-nginx
```

Stop:

```bash
docker stop test-nginx
```

Force stop:

```bash
docker kill test-nginx
```

Remove:

```bash
docker rm test-nginx
```

Force remove:

```bash
docker rm -f test-nginx
```

### Key idea

```text
docker run = create + start
```

---

# 11. Container Logs

Run:

```bash
docker run -d --name web nginx
```

View logs:

```bash
docker logs web
```

Follow:

```bash
docker logs -f web
```

Last 50 lines:

```bash
docker logs --tail 50 web
```

With timestamps:

```bash
docker logs -t web
```

Logs are one of the first places to look when an application fails.

---

# 12. Execute Commands Inside a Container

Start Ubuntu:

```bash
docker run -dit --name my-ubuntu ubuntu:24.04
```

Enter:

```bash
docker exec -it my-ubuntu bash
```

Inside:

```bash
cat /etc/os-release
```

```bash
ls
```

```bash
pwd
```

```bash
whoami
```

Exit:

```bash
exit
```

Run a single command:

```bash
docker exec my-ubuntu ls
```

```bash
docker exec my-ubuntu cat /etc/os-release
```

---

# 13. Understand `-it`

Consider:

```bash
docker exec -it my-ubuntu bash
```

- `-i` = interactive
- `-t` = allocate a terminal

Together:

```text
-it = interactive terminal
```

---

# 14. Port Mapping

Suppose Nginx listens on port 80 inside the container.

That does not automatically publish it on the host.

Use:

```bash
docker run -d --name web -p 8080:80 nginx
```

Meaning:

```text
Host port       Container port
    8080   --->     80
```

Open:

```text
http://localhost:8080
```

Check:

```bash
docker ps
```

You should see something like:

```text
0.0.0.0:8080->80/tcp
```
<img width="1133" height="354" alt="image" src="https://github.com/user-attachments/assets/6439cb89-ab88-4f1e-92d2-3bf58b3774e7" />

---

# 15. Port Mapping Practice

Run:

```bash
docker run -d --name web1 -p 8081:80 nginx
```

```bash
docker run -d --name web2 -p 8082:80 nginx
```

Open:

```text
http://localhost:8081
http://localhost:8082
```

### Hands-on Task 2

Launch three Nginx containers:

```text
web1 -> localhost:8081
web2 -> localhost:8082
web3 -> localhost:8083
```

Verify:

```bash
docker ps
```

Stop `web2` and verify that 8081 and 8083 still work.

---

# 16. Environment Variables

This section explains how to pass environment variables into a Docker container at runtime using the -e (or --env) flag. Environment variables are a standard way to configure applications without hardcoding values into images.

Run:

```bash
docker run --rm -e APP_NAME="DockerLab" ubuntu:24.04 env
```
<img width="805" height="314" alt="image" src="https://github.com/user-attachments/assets/946aba32-6243-4200-b938-53637218a2f9" />


You should see:

```text
APP_NAME=DockerLab
```

Another example:

```bash
docker run --rm -e NAME=Student ubuntu:24.04 sh -c 'echo Hello $NAME'
```

-e NAME=Student sets NAME inside the container.
sh -c 'echo Hello $NAME' runs a shell command that expands $NAME.
Single quotes '...' prevent your host shell from expanding $NAME — so the container's shell does the expansion.

Output:

```text
Hello Student
```

Example application configuration:

```bash
docker run -d \
  --name app \
  -e APP_ENV=development \
  -e DEBUG=true \
  nginx
```
<img width="805" height="190" alt="image" src="https://github.com/user-attachments/assets/946662d0-8fbc-461a-81d6-4c7c41f26ba3" />


---

# 17. Docker Inspect

Inspect a container:

```bash
docker inspect app
```

Get status:

```bash
docker inspect app --format '{{.State.Status}}'
```

Get IP address:

```bash
docker inspect app --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

Inspect an image:

```bash
docker image inspect nginx
```

---

# 18. Docker Stats

View resource usage:

```bash
docker stats
```

One container:

```bash
docker stats app
```

You can observe:

- CPU
- Memory
- Network I/O
- Block I/O
- Process count

Exit with:

```text
Ctrl + C
```

---

# 19. Docker Top

Show processes:

```bash
docker top app
```

---

# 20. Container Filesystem

Start:

```bash
docker run -dit --name filetest ubuntu:24.04
```

Enter:

```bash
docker exec -it filetest bash
```

Create:

```bash
echo "Hello Docker" > /hello.txt
```

Exit:

```bash
exit
```

Read:

```bash
docker exec filetest cat /hello.txt
```

Remove:

```bash
docker rm -f filetest
```

Create a new container:

```bash
docker run -dit --name filetest2 ubuntu:24.04
```

The file is gone.

### Lesson

Container writable storage is not a substitute for persistent storage.

---

# 21. Volumes

A named volume is storage that Docker itself creates and manages on the host (usually under /var/lib/docker/volumes/). You refer to it by name, and Docker handles the actual path.

Create:

```bash
docker volume create mydata
```

List:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect mydata
```

Use:

```bash
docker run -dit \
  --name volume-test \
  -v mydata:/data \
  ubuntu:24.04
```
<img width="805" height="251" alt="image" src="https://github.com/user-attachments/assets/6da6d2a9-74b7-480a-a55f-aeea4ff12cb2" />


Enter:

```bash
docker exec -it volume-test bash
```

Create:

```bash
echo "Persistent Docker Data" > /data/message.txt
```

Exit:

```bash
exit
```

Remove container:

```bash
docker rm -f volume-test
```

Create another container with the same volume:

```bash
docker run -dit \
  --name volume-test2 \
  -v mydata:/data \
  ubuntu:24.04
```

Read:

```bash
docker exec volume-test2 cat /data/message.txt
```

The data should still exist.

Takeaway: Even after destroying the original container, the data persisted because it lived in the volume, not the container's writable layer.

---

# 22. Bind Mounts

A bind mount maps an explicit path on your host machine directly into the container. You control exactly where the files live on the host.

Create a host directory:

```bash
mkdir -p ~/docker-lab/html
```

Create HTML:

```bash
echo "<h1>Hello from Host</h1>" > ~/docker-lab/html/index.html
```

Run:

```bash
docker run -d \
  --name bind-nginx \
  -p 8085:80 \
  -v ~/docker-lab/html:/usr/share/nginx/html:ro \
  nginx
```
<img width="805" height="251" alt="image" src="https://github.com/user-attachments/assets/e2e5075f-1473-47cf-93ee-cb2d54c9211b" />


Open:

```text
http://localhost:8085
```

Modify the host file:

```bash
echo "<h1>Changed from Host</h1>" > ~/docker-lab/html/index.html
```

Refresh the browser.

### Volume vs Bind Mount

```text
Named Volume:
Docker manages the storage.

Bind Mount:
You explicitly map a host path.
```
<img width="805" height="507" alt="image" src="https://github.com/user-attachments/assets/c1ef95d3-0ad3-46e6-b45b-511001a23e45" />

<img width="805" height="708" alt="image" src="https://github.com/user-attachments/assets/02adbd57-1423-4cfe-9b33-80400b5639ce" />


---

# 23. Docker Networks

Docker networking lets containers communicate with each other and with the outside world. Each network is an isolated virtual network.


List:

```bash
docker network ls
```
Three defaults exist out of the box:
bridge — default network for containers
host — container shares the host's network stack
none — no networking at all


Inspect default bridge:

```bash
docker network inspect bridge
```
Returns JSON with subnet info, gateway, and a list of connected containers. You'll see "Containers": {} if nothing is attached, or entries for running containers.

Create:

```bash
docker network create mynetwork
```

Run two containers:

```bash
docker run -dit --name container1 --network mynetwork ubuntu:24.04
```

```bash
docker run -dit --name container2 --network mynetwork ubuntu:24.04
```

Install ping tools if needed:

```bash
docker exec container1 bash
```

Inside:

```bash
apt update
apt install -y iputils-ping
```

Exit:

```bash
exit
```

On a user-defined network, containers can communicate using container names.

For example:

```bash
docker exec container1 ping -c 3 container2
```
Key point: On a user-defined network, Docker runs an embedded DNS server. Other containers can be reached by their container name — no need to look up IP addresses.

---

# 24. Default Bridge vs User-Defined Network

Create:

```bash
docker network create app-network
```

Run:

```bash
docker run -d --name web --network app-network nginx
```

Run a client:

```bash
docker run -dit --name client --network app-network ubuntu:24.04
```

The containers can communicate through the Docker network.

---

<img width="668" height="483" alt="image" src="https://github.com/user-attachments/assets/5a15dc80-df73-4de4-b78c-4012ca480a8c" />


# 25. Dockerfile: Build Your Own Image

A Dockerfile contains image-building instructions.

Common instructions:

```dockerfile
FROM
WORKDIR
COPY
ADD
RUN
CMD
ENTRYPOINT
EXPOSE
ENV
ARG
USER
```

The most commonly used instructions in Dockerfile are:
FROM: specifies the base image for the build
RUN: runs a command to install software or make other changes to the image
COPY: copies files or directories from the host machine to the image
ENV: sets environment variables
EXPOSE: specifies the ports that the container will listen on
CMD: specifies the command that will be run when a container is started from the image
ENTRYPOINT: instruction sets the command that will be executed when the container is started from the image.
Unlike the CMD instruction, the ENTRYPOINT instruction does not get overridden when additional command-line arguments  are passed to the docker run command


<img width="736" height="364" alt="image" src="https://github.com/user-attachments/assets/fa4189a9-9786-4ae9-b1df-8ed4f212ccaa" />


---

# 26. First Dockerfile

Create:

```bash
mkdir docker-first-image
cd docker-first-image
```

Create `Dockerfile`:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

CMD ["curl", "https://example.com"]
```

Build:

```bash
docker build -t my-first-image .
```
<img width="752" height="455" alt="image" src="https://github.com/user-attachments/assets/9c9d78e5-6184-4a09-888e-9ab515e99693" />


Check:

```bash
docker images
```

Run:

```bash
docker run --rm my-first-image
```

---

# 27. Understand the Dockerfile

## FROM

```dockerfile
FROM ubuntu:24.04
```
<img width="632" height="366" alt="image" src="https://github.com/user-attachments/assets/c8adbf82-9419-44f1-9919-7b92d5c8e7fa" />


Specifies the base image.

## RUN

```dockerfile
RUN apt-get update
```

Runs during image building.

## CMD

```dockerfile
CMD ["curl", "https://example.com"]
```

Specifies the default command when the container starts.

<img width="809" height="229" alt="image" src="https://github.com/user-attachments/assets/650cc885-1cb1-4577-ae40-57003046d9f7" />


---

# 28. Docker Build Context

When you run:

```bash
docker build -t myimage .
```

The final `.` means:

```text
Use the current directory as the build context.
Send everything in the current directory to the Docker daemon to be used as the build context.
```

Files used by `COPY` must be inside the build context.

---

# 29. Build a Simple Web Image

Create:

```bash
mkdir docker-web
cd docker-web
```

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Lab</title>
</head>
<body>
    <h1>Hello from My Docker Image!</h1>
    <p>This page is running inside a Docker container.</p>
</body>
</html>
```

Create `Dockerfile`:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build:

```bash
docker build -t docker-web:v1 .
```

Run:

```bash
docker run -d \
  --name docker-web \
  -p 8080:80 \
  docker-web:v1
```

Open:

```text
http://localhost:8080
```

---

# 30. EXPOSE vs `-p`

Dockerfile:

```dockerfile
EXPOSE 80
```

does not publish the port to the host by itself.

You still need:

```bash
docker run -p 8080:80 ...
```

Think:

```text
EXPOSE
    |
    | Documents the intended container port
    |
-p
    |
    | Publishes/maps the host port
```

---

# 31. Dockerfile `COPY`

Suppose:

```text
project/
├── Dockerfile
├── app.py
└── requirements.txt
```

Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

---

# 32. WORKDIR

Instead of:

```dockerfile
RUN mkdir /app
RUN cd /app
```

use:

```dockerfile
WORKDIR /app
```

Then:

```dockerfile
COPY app.py .
```

means:

```text
/app/app.py
```

---

# 33. RUN vs CMD

### RUN

Executed during image building:

```dockerfile
RUN pip install flask
```

### CMD

Executed by default when the container starts:

```dockerfile
CMD ["python", "app.py"]
```

Mental model:

```text
docker build
    |
    +-- RUN

docker run
    |
    +-- CMD
```

---

# 34. CMD vs ENTRYPOINT

CMD:

```dockerfile
CMD ["python", "app.py"]
```

provides a default command that can be overridden.

ENTRYPOINT:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

makes the image behave more like a fixed executable.

For introductory application containers, understand CMD first.

---

# 35. Docker Image Layers

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

A simplified mental model:

```text
Base Python image
      +
Application dependency layer
      +
Application code layer
      +
Runtime configuration
```

Docker can reuse cached build layers.

This is why Dockerfile ordering matters.

---

# 36. `.dockerignore`

Create:

```bash
nano .dockerignore
```

Example:

```text
.git
.gitignore
__pycache__
*.pyc
.env
node_modules
venv
.venv
```

It prevents unnecessary files from being sent to the Docker build context.

Never casually copy secrets into an image.

---

# 37. Flask Application: Real Hands-On Project

Project:

```text
flask-docker/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

Create:

```bash
mkdir flask-docker
cd flask-docker
```

---

# 38. Create Flask Application

Create `app.py`:

```python
from flask import Flask
import os

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <h1>Hello from Docker!</h1>
    <p>This Flask application is running inside a Docker container.</p>
    """

@app.route("/info")
def info():
    return {
        "message": "Flask application inside Docker",
        "hostname": os.uname().nodename,
        "environment": os.getenv("APP_ENV", "not-set")
    }

@app.route("/hello/<name>")
def hello(name):
    return f"<h1>Hello {name}!</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### Important

```python
host="0.0.0.0"
```

allows Docker's published port to reach Flask.

---

# 39. Create Requirements

Create `requirements.txt`:

```text
Flask
```

---

# 40. Create Flask Dockerfile

Create `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# 41. Create `.dockerignore`

```text
__pycache__
*.pyc
.venv
venv
.git
.env
```

---

# 42. Build Flask Image

From the project directory:

```bash
docker build -t flask-docker:v1 .
```

Check:

```bash
docker images
```

---

# 43. Run Flask Container

```bash
docker run -d \
  --name flask-app \
  -p 5000:5000 \
  flask-docker:v1
```

Check:

```bash
docker ps
```

Open:

```text
http://localhost:5000
```

Open:

```text
http://localhost:5000/info
```

---

# 44. Test Flask with curl

```bash
curl http://localhost:5000
```

```bash
curl http://localhost:5000/info
```

```bash
curl http://localhost:5000/hello/Ali
```

---

# 45. Add Environment Variable

Remove old container:

```bash
docker rm -f flask-app
```

Run:

```bash
docker run -d \
  --name flask-app \
  -p 5000:5000 \
  -e APP_ENV=development \
  flask-docker:v1
```

Test:

```bash
curl http://localhost:5000/info
```

The response should show:

```text
"environment": "development"
```

---

# 46. Modify and Rebuild

Modify `app.py`.

Build a new version:

```bash
docker build -t flask-docker:v2 .
```

Remove old container:

```bash
docker rm -f flask-app
```

Run:

```bash
docker run -d \
  --name flask-app \
  -p 5000:5000 \
  flask-docker:v2
```

Test:

```bash
curl http://localhost:5000/hello/Ali
```

---

# 47. Image vs Container

Suppose:

```text
flask-docker:v2
```

is the image.

You can create multiple containers:

```bash
docker run -d --name flask1 -p 5001:5000 flask-docker:v2
```

```bash
docker run -d --name flask2 -p 5002:5000 flask-docker:v2
```

```bash
docker run -d --name flask3 -p 5003:5000 flask-docker:v2
```

Mental model:

```text
Image
  |
  +---- Container flask1
  |
  +---- Container flask2
  |
  +---- Container flask3
```

Image = template.

Container = instance.

---

# 48. Tagging Images

List:

```bash
docker images
```

Tag:

```bash
docker tag flask-docker:v2 YOUR_DOCKERHUB_USERNAME/flask-docker:v2
```

The same image can have multiple tags.

---

# 49. Docker Hub

Login:

```bash
docker login
```

Tag:

```bash
docker tag flask-docker:v2 YOUR_DOCKERHUB_USERNAME/flask-docker:v2
```

Push:

```bash
docker push YOUR_DOCKERHUB_USERNAME/flask-docker:v2
```

On another machine:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/flask-docker:v2
```

Run:

```bash
docker run -d \
  --name flask-from-hub \
  -p 5000:5000 \
  YOUR_DOCKERHUB_USERNAME/flask-docker:v2
```

Replace `YOUR_DOCKERHUB_USERNAME` with the actual Docker Hub username.

---

# 50. Docker Compose

Real applications often contain multiple services:

```text
Web Application
       |
       v
    Database
```

Compose lets us define the application in YAML.

Check:

```bash
docker compose version
```

---

# 51. Simple Compose Example

Create:

```bash
mkdir compose-demo
cd compose-demo
```

Create `compose.yaml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs
```

Follow:

```bash
docker compose logs -f
```

Stop and remove:

```bash
docker compose down
```

---

# 52. Compose Flask Application

Project:

```text
flask-compose/
├── app.py
├── requirements.txt
├── Dockerfile
└── compose.yaml
```

Use the Flask files from the previous section.

Create `compose.yaml`:

```yaml
services:
  web:
    build: .
    container_name: flask-compose-app
    ports:
      - "5000:5000"
    environment:
      APP_ENV: compose
```

Build and start:

```bash
docker compose up -d --build
```

Check:

```bash
docker compose ps
```

Test:

```bash
curl http://localhost:5000/info
```

Stop:

```bash
docker compose down
```

---

# 53. Flask + Redis with Compose

This demonstrates a realistic multi-container application.

Project:

```text
flask-redis/
├── app.py
├── requirements.txt
├── Dockerfile
└── compose.yaml
```

`requirements.txt`:

```text
Flask
redis
```

`app.py`:

```python
from flask import Flask
import redis
import os

app = Flask(__name__)

redis_host = os.getenv("REDIS_HOST", "redis")
r = redis.Redis(host=redis_host, port=6379, decode_responses=True)

@app.route("/")
def home():
    try:
        count = r.incr("visits")
        return f"<h1>Visits: {count}</h1>"
    except Exception as e:
        return f"<h1>Redis error: {e}</h1>"

@app.route("/health")
def health():
    try:
        r.ping()
        return {"status": "ok", "redis": "connected"}
    except Exception as e:
        return {"status": "error", "redis": str(e)}, 500

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

`compose.yaml`:

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      REDIS_HOST: redis
    depends_on:
      - redis

  redis:
    image: redis:alpine
```

Start:

```bash
docker compose up -d --build
```

Check:

```bash
docker compose ps
```

Test:

```bash
curl http://localhost:5000/
```

Run several times:

```bash
curl http://localhost:5000/
curl http://localhost:5000/
curl http://localhost:5000/
```

Health:

```bash
curl http://localhost:5000/health
```

Logs:

```bash
docker compose logs web
```

---

# 54. Important Compose Networking Concept

The Flask application connects to:

```text
redis
```

not:

```text
localhost
```

Inside the Flask container:

```text
localhost = Flask container itself
```

Redis is another container.

Compose provides service-name DNS:

```text
web  ---> redis
```

So:

```python
redis.Redis(host="redis", port=6379)
```

works.

---

# 55. `depends_on` Limitation

This:

```yaml
depends_on:
  - redis
```

helps with startup ordering, but it does not guarantee that Redis is fully ready to accept connections.

Production applications should use health checks and/or retry logic where appropriate.

---

# 56. Docker Health Checks

Example:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1
```

For this example, the image must contain `curl`.

Inspect:

```bash
docker ps
```

or:

```bash
docker inspect CONTAINER_NAME
```

---

# 57. Resource Limits

Memory:

```bash
docker run -d \
  --name limited-app \
  --memory="256m" \
  nginx
```

CPU:

```bash
docker run -d \
  --name cpu-limited-app \
  --cpus="0.5" \
  nginx
```

Monitor:

```bash
docker stats
```

---

# 58. Restart Policies

Run:

```bash
docker run -d \
  --name restart-demo \
  --restart unless-stopped \
  nginx
```

Common policies:

```text
no
always
on-failure
unless-stopped
```

Update:

```bash
docker update --restart unless-stopped restart-demo
```

---

# 59. Copy Files Between Host and Container

Create:

```bash
echo "Hello Container" > host.txt
```

Copy into:

```bash
docker cp host.txt restart-demo:/tmp/host.txt
```

Check:

```bash
docker exec restart-demo cat /tmp/host.txt
```

Copy out:

```bash
docker cp restart-demo:/etc/nginx/nginx.conf .
```

---

# 60. View Container Changes

```bash
docker diff restart-demo
```

This shows filesystem changes made inside the container.

---

# 61. Save and Load Images

Save:

```bash
docker save -o flask-docker.tar flask-docker:v2
```

Load elsewhere:

```bash
docker load -i flask-docker.tar
```

This preserves the Docker image.

---

# 62. Image History

```bash
docker image history flask-docker:v2
```

Students should observe the image's layers.

---

# 63. Build Without Cache

```bash
docker build --no-cache -t flask-docker:clean .
```

Useful when debugging Docker build caching.

---

# 64. Build with Detailed Output

```bash
docker build --progress=plain -t flask-docker:v3 .
```

Useful when students need to understand each build step.

---

# 65. Debugging Workflow

When an application does not work:

## Step 1: Is the container running?

```bash
docker ps
```

If not:

```bash
docker ps -a
```

## Step 2: Read logs

```bash
docker logs CONTAINER
```

## Step 3: Inspect

```bash
docker inspect CONTAINER
```

## Step 4: Check ports

```bash
docker port CONTAINER
```

## Step 5: Enter the container

```bash
docker exec -it CONTAINER sh
```

or:

```bash
docker exec -it CONTAINER bash
```

## Step 6: Processes

```bash
docker top CONTAINER
```

## Step 7: Resources

```bash
docker stats CONTAINER
```

---

# 66. Common Problems

## Problem 1: Port already in use

Possible error:

```text
bind: address already in use
```

Find the process:

```bash
sudo ss -ltnp | grep :5000
```

Or use another host port:

```bash
docker run -d -p 5001:5000 flask-docker:v2
```

---

## Problem 2: Container exits immediately

Check:

```bash
docker ps -a
```

Then:

```bash
docker logs CONTAINER
```

A container normally lives while its main process is running.

```text
Main process running -> container running
Main process exits   -> container exits
```

---

## Problem 3: Flask works inside container but not from browser

Check:

```python
app.run(host="0.0.0.0", port=5000)
```

Then:

```bash
docker run -p 5000:5000 ...
```

Do not bind the application only to:

```text
127.0.0.1
```

inside the container when you need published external access.

---

## Problem 4: `localhost` confusion

From host:

```text
localhost:5000
```

means the host machine.

Inside a container:

```text
localhost
```

means that container.

This is one of the most important Docker networking concepts.

---

# 67. Docker Command Cheat Sheet

## Information

```bash
docker --version
docker version
docker info
docker system df
```

## Images

```bash
docker pull IMAGE
docker images
docker image ls
docker image inspect IMAGE
docker image history IMAGE
docker image rm IMAGE
docker tag SOURCE TARGET
docker push IMAGE
```

## Containers

```bash
docker run IMAGE
docker run -d IMAGE
docker ps
docker ps -a
docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker kill CONTAINER
docker rm CONTAINER
docker rm -f CONTAINER
```

## Debugging

```bash
docker logs CONTAINER
docker logs -f CONTAINER
docker inspect CONTAINER
docker exec -it CONTAINER bash
docker top CONTAINER
docker stats CONTAINER
docker port CONTAINER
docker diff CONTAINER
```

## Copy

```bash
docker cp FILE CONTAINER:/PATH
docker cp CONTAINER:/PATH FILE
```

## Volumes

```bash
docker volume ls
docker volume create NAME
docker volume inspect NAME
docker volume rm NAME
```

## Networks

```bash
docker network ls
docker network create NAME
docker network inspect NAME
docker network connect NETWORK CONTAINER
docker network disconnect NETWORK CONTAINER
docker network rm NAME
```

## Compose

```bash
docker compose up
docker compose up -d
docker compose up -d --build
docker compose ps
docker compose logs
docker compose logs -f
docker compose exec SERVICE bash
docker compose stop
docker compose start
docker compose restart
docker compose down
```

---

# 68. Docker Cleanup

Show disk usage:

```bash
docker system df
```

Remove stopped containers:

```bash
docker container prune
```

Remove unused networks:

```bash
docker network prune
```

Remove unused images:

```bash
docker image prune
```

More aggressive cleanup:

```bash
docker system prune
```

Include unused volumes:

```bash
docker system prune --volumes
```

> **Warning:** Cleanup commands can delete resources. Students should understand the command before running it.

---

# 69. Docker System Information

```bash
docker info
```

This provides information about:

- Docker version
- Containers
- Images
- Storage driver
- CPUs
- Memory
- Operating system
- Docker root directory
- Plugins

---

# 70. Docker Storage Model

Simplified:

```text
Image
  |
  | read-only layers
  v
Container
  |
  | writable layer
  v
Temporary changes
```

For persistent data:

```text
Container
    |
    v
Volume
    |
    v
Persistent data
```

---

# 71. Docker Security Basics

## Do not put secrets in Dockerfiles

Avoid:

```dockerfile
ENV PASSWORD=mysecret
```

Avoid:

```dockerfile
COPY .env .
```

Secrets can become visible through image history or layers.

## Use `.dockerignore`

Example:

```text
.env
.git
*.key
*.pem
```

## Avoid unnecessary root privileges

For applications, consider:

```dockerfile
USER appuser
```

## Use appropriate base images

For example:

```text
python:3.12-slim
```

when a slim image is suitable.

## Keep images and dependencies updated

---

# 72. Flask as a Non-Root User

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN useradd --create-home appuser

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t flask-secure:v1 .
```

Run:

```bash
docker run -d --name flask-secure -p 5000:5000 flask-secure:v1
```

Check:

```bash
docker exec flask-secure whoami
```

Expected:

```text
appuser
```

---
