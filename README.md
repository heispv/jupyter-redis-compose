# Jupyter-Redis Docker Integration

This repository provides instructions on how to set up a Docker environment that integrates Jupyter notebooks with the Redis database. The setup involves creating a custom Docker image based on the `jupyter/minimal-notebook` image with the Redis library pre-installed.

## Getting Started

### Step 1: Pull the Base Docker Image

Start by pulling the official `jupyter/minimal-notebook` image from Docker Hub:

```bash
docker pull jupyter/minimal-notebook
```

### Step 2: Create a Custom Docker Image

Create a Dockerfile to build a custom image that includes the Redis library:

```dockerfile
FROM jupyter/minimal-notebook
RUN pip install redis
```

Build the custom image using the following command:

```bash
docker build -t jupyter-redis .
```

### Step 3: Set Up a Docker Network

Create a bridge network named `bdp2-net` for your Docker containers:

```bash
docker network create bdp2-net
```

### Step 4: Run the Jupyter Container

Launch the Jupyter container using the custom image:

```bash
docker run -d --rm --name my_jupyter -v ./work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="admin" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" jupyter-redis
```

**Note:**
- Adjust your AWS security group settings to allow traffic on the required port if deploying on an AWS instance.
- Steps will be automated with Docker Compose in the future.

### Step 5: Define the Docker Compose Configuration

Since the Redis image does not require special configuration, you can define both services in a single `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  redis:
    image: redis
    container_name: my_redis
    volumes:
      - ./work:/data
    networks:
      - bdp2-net
    user: "1000"
    command: redis-server

  jupyter:
    build: .
    container_name: my_jupyter
    volumes:
      - ./work:/home/jovyan
    ports:
      - "80:8888"
    networks:
      - bdp2-net
    environment:
      - JUPYTER_ENABLE_LAB=yes
      - JUPYTER_TOKEN=admin
      - CHOWN_HOME=yes
      - CHOWN_HOME_OPTS=-R
    user: root

networks:
  bdp2-net:
    driver: bridge
```

Running `docker compose up` will start both containers, with the Jupyter notebook accessible and configured with the Redis library.

## Directory Structure

Here's the layout of the project directory:

```
jupyter-redis-compose/
├── docker-compose.yml
├── docker/
│   └── Dockerfile
└── work/
```

**Note:** The `work` directory is listed in the `.gitignore` file to prevent it from being tracked by Git.
