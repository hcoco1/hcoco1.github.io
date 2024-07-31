---
title: Common Docker Commands Explained
author: hcoco1
date: 2024-07-31 14:10:00 +0800
categories: [Programming, Tools]
tags: [docker]
render_with_liquid: false
---

Docker has become an essential tool in the world of software development and deployment, offering a streamlined way to manage applications and their dependencies. Whether you're a beginner or an experienced user, understanding the most common Docker commands is crucial for efficient container management. This post will explore the key Docker commands and their functionalities.

## Basic Docker Commands

### 1. `docker run`

**Purpose:** Create and start a container from an image.

**Usage:**
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Example:**
```bash
docker run -d -p 80:80 nginx
```
This command runs an Nginx container in detached mode (`-d`) and maps port 80 of the host to port 80 of the container.

### 2. `docker pull`

**Purpose:** Download an image from a Docker registry.

**Usage:**
```bash
docker pull IMAGE[:TAG]
```

**Example:**
```bash
docker pull ubuntu:latest
```
This command pulls the latest Ubuntu image from Docker Hub.

### 3. `docker ps`

**Purpose:** List running containers.

**Usage:**
```bash
docker ps [OPTIONS]
```

**Example:**
```bash
docker ps
```
This command lists all running containers.

### 4. `docker stop`

**Purpose:** Stop a running container.

**Usage:**
```bash
docker stop CONTAINER
```

**Example:**
```bash
docker stop my-container
```
This command stops a container named `my-container`.

### 5. `docker rm`

**Purpose:** Remove a container.

**Usage:**
```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Example:**
```bash
docker rm my-container
```
This command removes a container named `my-container`.

### 6. `docker rmi`

**Purpose:** Remove an image.

**Usage:**
```bash
docker rmi IMAGE [IMAGE...]
```

**Example:**
```bash
docker rmi ubuntu:latest
```
This command removes the `ubuntu:latest` image.

### 7. `docker exec`

**Purpose:** Run a command in a running container.

**Usage:**
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**Example:**
```bash
docker exec -it my-container /bin/bash
```
This command opens an interactive terminal in the `my-container`.

## Intermediate Docker Commands

### 1. `docker build`

**Purpose:** Build an image from a Dockerfile.

**Usage:**
```bash
docker build [OPTIONS] PATH | URL | -
```

**Example:**
```bash
docker build -t my-image:latest .
```
This command builds an image named `my-image` with the `latest` tag from the current directory.

### 2. `docker logs`

**Purpose:** Fetch the logs of a container.

**Usage:**
```bash
docker logs [OPTIONS] CONTAINER
```

**Example:**
```bash
docker logs my-container
```
This command fetches the logs from `my-container`.

### 3. `docker network ls`

**Purpose:** List all networks.

**Usage:**
```bash
docker network ls
```

**Example:**
```bash
docker network ls
```
This command lists all Docker networks.

### 4. `docker volume ls`

**Purpose:** List all volumes.

**Usage:**
```bash
docker volume ls
```

**Example:**
```bash
docker volume ls
```
This command lists all Docker volumes.

### 5. `docker inspect`

**Purpose:** Return low-level information on Docker objects.

**Usage:**
```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

**Example:**
```bash
docker inspect my-container
```
This command returns detailed information about `my-container`.

## Advanced Docker Commands

### 1. `docker-compose up`

**Purpose:** Create and start containers using Docker Compose.

**Usage:**
```bash
docker-compose up [OPTIONS] [SERVICE...]
```

**Example:**
```bash
docker-compose up -d
```
This command starts the services defined in `docker-compose.yml` in detached mode.

### 2. `docker swarm init`

**Purpose:** Initialize a swarm.

**Usage:**
```bash
docker swarm init [OPTIONS]
```

**Example:**
```bash
docker swarm init
```
This command initializes the current node as a swarm leader.

### 3. `docker secret create`

**Purpose:** Create a secret.

**Usage:**
```bash
docker secret create [OPTIONS] SECRET DATA|- 
```

**Example:**
```bash
echo "my_secret_password" | docker secret create db_password -
```
This command creates a Docker secret named `db_password`.

### 4. `docker stack deploy`

**Purpose:** Deploy a new stack or update an existing stack.

**Usage:**
```bash
docker stack deploy [OPTIONS] STACK
```

**Example:**
```bash
docker stack deploy -c docker-compose.yml my_stack
```
This command deploys a stack named `my_stack` using the `docker-compose.yml` file.

### 5. `docker prune`

**Purpose:** Remove unused data.

**Usage:**
```bash
docker system prune [OPTIONS]
```

**Example:**
```bash
docker system prune -a
```
This command removes all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.

