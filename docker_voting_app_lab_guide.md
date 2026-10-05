# Docker Containers & Docker Compose Lab Guide: Example Voting App

This hands-on lab guide teaches you how to containerize and orchestrate a multi-tier distributed application (**Example Voting App**) using native Docker commands (manual networking, building, and running) and then automate the entire setup using **Docker Compose**.

## 🏗️ Architecture Overview

The Example Voting App consists of 5 multi-container components written in different languages and using different databases:

1. **Vote (Python / Flask):** A front-end web app that lets users vote between two options (e.g., Cats vs. Dogs).

2. **Redis (In-memory data store):** A message queue that collects incoming votes.

3. **Worker (.NET Core):** A background worker that consumes votes from Redis and stores them in PostgreSQL.

4. **PostgreSQL (Database):** A persistent relational database that stores the final vote counts.

5. **Result (Node.js / Express):** A front-end web app that displays the results of the voting in real-time.

## Part 1: Setting Up the Repository

Before running any Docker commands, make sure you have the source code cloned on your local machine:

```
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app

```

## Part 2: Manual Container Deployment (Step-by-Step)

To manually run each container and link them together, we use a **custom user-defined Docker Network**. This allows containers to automatically find and communicate with each other using their container names as hostnames via Docker's built-in DNS resolution (replacing the legacy `--link` flag).

### Step 1: Create a Custom Docker Network

Create a bridge network so all five containers can securely talk to one another on an isolated virtual network.

```
docker network create voting-net

```

* **`docker network create`**: Tells Docker to create a new network.

* **`voting-net`**: The user-defined name given to the network bridge.

### Step 2: Set Up the Persistent Database (`db`)

Run the PostgreSQL container first. It requires an external volume/storage strategy or environment configuration to track credentials, and it needs to join our custom network.

```
docker run -d \
  --name db \
  --network voting-net \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  postgres:15-alpine

```

* **`-d` (Detached mode)**: Runs the container in the background and prints the container ID.

* **`--name db`**: Assigns a static hostname/name (`db`) to this container so other containers (like the worker) can reference it directly.

* **`--network voting-net`**: Connects this container to our custom bridge network.

* **`-e ...`**: Sets environment variables inside the container to configure the PostgreSQL username and password.

* **`postgres:15-alpine`**: Uses the lightweight Alpine-based official PostgreSQL version 15 image.

### Step 3: Set Up the Message Queue (`redis`)

Run the Redis container. The Python web app will push votes here, and the .NET worker will pull votes from here.

```
docker run -d \
  --name redis \
  --network voting-net \
  redis:alpine

```

* **`--name redis`**: Names the container `redis`, allowing the vote app and worker to connect using `redis` as their hostname.

* **`redis:alpine`**: Pulls the lightweight, optimized Alpine Linux image for Redis.

### Step 4: Build and Run the Voting Web App (`vote`)

The vote app is custom-built using Python. You must build its Docker image locally from the source code located in the `./vote` directory.

Make sure you are in the root directory of the repository (`example-voting-app`), then execute:

```
# 1. Build the image locally
docker build -t voting-app-vote ./vote

# 2. Run the container and expose port 8080 to your machine
docker run -d \
  --name vote \
  --network voting-net \
  -p 8080:80 \
  voting-app-vote

```

* **`docker build -t voting-app-vote ./vote`**: Reads the `Dockerfile` inside the `./vote` folder, packages the source code and dependencies, and tags (`-t`) the final image as `voting-app-vote`.

* **`-p 8080:80`**: Maps port `8080` on your host machine to port `80` inside the container (where the Flask web server is listening). This allows you to access it via `http://localhost:8080`.

### Step 5: Build and Run the Result Web App (`result`)

The result frontend is built using Node.js. Navigate or point to the `./result` directory to build and run it.

```
# 1. Build the image locally
docker build -t voting-app-result ./result

# 2. Run the container and expose port 8081 to your machine
docker run -d \
  --name result \
  --network voting-net \
  -p 8081:80 \
  voting-app-result

```

* **`docker build -t voting-app-result ./result`**: Builds a production-ready container image for the Node.js application.

* **`-p 8081:80`**: Exposes port `8081` on your computer so you can view live graph updates at `http://localhost:8081`.

### Step 6: Build and Run the Background Worker (`worker`)

The worker connects the Redis queue to the PostgreSQL database. It runs continuously in the background processing jobs. **It does not expose any public ports** because end-users do not interact with it directly.

```
# 1. Build the image locally
docker build -t voting-app-worker ./worker

# 2. Run the container
docker run -d \
  --name worker \
  --network voting-net \
  voting-app-worker

```

* **`docker build -t voting-app-worker ./worker`**: Compiles and packages the .NET Core worker image.

* **No `-p` flag**: Since it only talks to internal services (`redis` and `db` via `voting-net`), no ports need to be mapped to the host machine.

### 🛠️ Verification & Troubleshooting Manual Setup

1. **Check running containers:**

   ```
   docker ps
   
   ```

   *You should see all 5 containers active (`vote`, `result`, `worker`, `redis`, `db`).*

2. **Access the Web Applications:**

   * **Voting UI:** Open your browser and go to <http://localhost:8080> to cast a vote.

   * **Results UI:** Open a separate tab/window and go to <http://localhost:8081> to watch the vote counts update live.

3. **How Network Linking Works:**
   Because every container was started on `--network voting-net`, the application code resolves other containers seamlessly using their name strings (`db`, `redis`) as hostnames without manual IP address lookups.

## Part 3: Automating Everything with Docker Compose

Running 5 separate build commands and 5 separate `docker run` commands with long arguments is repetitive and prone to error. **Docker Compose** allows us to define multi-container applications in a single declarative YAML file (`docker-compose.yml`) and manage them with a single command.

### 1. Inspecting the `docker-compose.yml` File

The official repository comes with a pre-configured `docker-compose.yml` file (or `docker-compose-app.yml`). It looks structurally similar to this snippet:

```
version: '3'
services:
  redis:
    image: redis:alpine
    networks:
      - voting-net

  db:
    image: postgres:15-alpine
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    networks:
      - voting-net

  vote:
    build: ./vote
    command: python app.py
    volumes:
      - ./vote:/code
    ports:
      - "8080:80"
    networks:
      - voting-net

  result:
    build: ./result
    command: node server.js
    volumes:
      - ./result:/app
    ports:
      - "8081:80"
    networks:
      - voting-net

  worker:
    build: ./worker
    networks:
      - voting-net

volumes:
  db-data:

networks:
  voting-net:

```

### 2. How to Run the Docker Compose File

Open your terminal inside the root directory containing the `docker-compose.yml` file and use the following core commands:

#### **Start the entire application stack (Build & Run in Background)**

```
docker compose up -d --build

```

* **`up`**: Aggregates, creates networks, builds images, creates volumes, and starts all services defined in the configuration file.

* **`-d`**: Runs the containers in detached mode (in the background).

* **`--build`**: Forces Docker Compose to rebuild local image sources (helpful if you modify source code in `/vote`, `/result`, or `/worker`).

#### **View real-time application logs**

```
docker compose logs -f

```

* **`-f` (follow)**: Streams live logs from all containers concurrently, allowing you to watch database connections, vote submissions, and worker actions in real time. Press `Ctrl + C` to exit logs.

#### **Check the status of services**

```
docker compose ps

```

#### **Stop and remove the application stack**

To stop all containers and clean up the network resources created by Compose:

```
docker compose down

```

*(Note: If you want to completely purge persistent database volumes as well, you can run `docker compose down -v`)*