Got it, Mahesh! I'll take your raw notes as-is, organize them neatly with proper headings and subheadings, add useful definitions and explanations, and include 10-15 practical scenario-based questions at the end. Then I will generate a clean, visually appealing Markdown (`.MD`) file for you.

Here’s the plan for the structure and enhancements:

---

# **Docker Container Commands**

---

## 1. Creating Containers

* **`docker create`**: Creates a container but **does not start** it.
* **`docker run`**: Creates and **starts** the container immediately.

### Examples:

```bash
docker create -p 8080:8080 --name javawebapp kkeducation12345/java-web-app
docker start javawebapp
docker run -d -p 8080:8080 --name javawebapp1 kkeducation12345/java-web-app
```

---

## 2. Listing Containers

* List **running** containers:

  ```bash
  docker ps
  docker container ls
  ```
* List **all** containers (running + stopped):

  ```bash
  docker ps -a
  docker container ls -a
  ```

---

## 3. Starting and Stopping Containers

* Start container:

  ```bash
  docker start <container_id_or_name>
  ```
* Stop container:

  ```bash
  docker stop <container_id_or_name>
  ```

---

## 4. Deleting Containers

* Delete a **stopped** container:

  ```bash
  docker rm <container_id_or_name>
  ```
* Force delete a **running** container:

  ```bash
  docker rm -f <container_id_or_name>
  ```

### Delete multiple containers:

```bash
docker rm -f <cid1> <cid2> ...
docker rm -f $(docker ps -q)       # Remove all running containers
docker rm -f $(docker ps -aq)      # Remove all containers (running + stopped)
```

---

## 5. Removing Containers by Status

* Remove all **stopped** containers:

  ```bash
  docker container prune
  docker rm -f $(docker ps --filter status="exited" -aq)
  ```

* Remove all **paused** containers:

  ```bash
  docker rm -f $(docker ps --filter status="paused")
  ```

---

## 6. Managing Container Names and Restarting

* Rename a container:

  ```bash
  docker rename <old_name_or_id> <new_name>
  ```
* Restart a container:

  ```bash
  docker restart <container_id_or_name>
  ```

---

## 7. Pausing and Unpausing Containers

* Pause container process:

  ```bash
  docker pause <container_id_or_name>
  ```
* Resume paused container:

  ```bash
  docker unpause <container_id_or_name>
  ```

---

## 8. Running Containers Interactively

* Create and start an Ubuntu container interactively:

  ```bash
  docker run -it --name test ubuntu /bin/bash
  ```

---

## 9. Debugging Containers

* View logs:

  ```bash
  docker logs <container_id_or_name>
  ```
* View last N lines:

  ```bash
  docker logs --tail <number> <container_id_or_name>
  ```
* Follow logs (real-time streaming):

  ```bash
  docker logs -f <container_id_or_name>
  ```

---

## 10. Executing Commands Inside Containers

* Get shell inside container:

  ```bash
  docker exec -it <container_id_or_name> /bin/bash
  ```
* Execute specific command:

  ```bash
  docker exec <container_id> ls
  ```

---

## 11. Inspecting Running Processes in Container

* Show running processes inside container:

  ```bash
  docker top <container_id_or_name>
  ```

---

## 12. Monitoring Resource Usage

* Show real-time resource stats (CPU, RAM):

  ```bash
  docker stats <container_id_or_name>
  ```

### Custom Memory Allocation Example:

```bash
docker run --memory="512m" -d -p 8082:8080 --name javawebapp2 kkeducation12345/java-web-app:1
```

> If memory is too low (e.g. 64m), container will be killed (`OOMKilled`):

```bash
docker inspect <container_id>   # Check if OOMKilled
```

---

## 13. Copying Files Between Host and Container

* Copy file from container to host:

  ```bash
  docker cp <container_name>:/path/in/container/file.log /path/on/host/
  ```
* Copy file from host to container:

  ```bash
  docker cp <file_on_host> <container_name>:/path/in/container/
  ```

### Difference between `COPY` and `docker cp`

* `COPY`: Dockerfile instruction used **while building images**.
* `docker cp`: CLI command used to **copy files/folders between running containers and host**.

---

## 14. Inspecting Container Differences

* Show file system changes in container:

  ```bash
  docker diff <container_id_or_name>
  ```

---

## 15. Committing Container Changes

* Create a new image from container state:

  ```bash
  docker commit <container_id_or_name> <new_image_name>:<tag>
  ```

---

## 16. Inspecting Containers

* Basic inspection:

  ```bash
  docker inspect <container_id_or_name>
  ```
* Inspect with format to extract specific info:

```bash
docker inspect --format "{{.NetworkSettings.IPAddress}}" mavenapp

docker inspect --format "{{.State.Pid}}" 0d2dab452ceb

docker inspect --format "{{.NetworkSettings.IPAddress}},{{.State.Status}}" 0d2dab452ceb

docker inspect --format "IP Address: {{.NetworkSettings.Networks.bridge.IPAddress}}, Name: {{.Name}}" mavenapp
```

---

# **Key Concepts & Definitions**

---

### What is the difference between `docker create` and `docker run`?

* `docker create`: Creates a container but does NOT start it.
* `docker run`: Creates and starts the container immediately.

---

### What is port publish or port mapping in Docker?

* It maps a port on the host machine to a port inside the container, enabling access to container services via the host.

---

### Difference between `docker stop` and `docker kill`?

* `docker stop`: Sends `SIGTERM` first, then `SIGKILL` after a grace period, allowing graceful shutdown.
* `docker kill`: Sends `SIGKILL` immediately, forcefully terminating the container without grace.

---

# **Scenario-Based Questions**

1. How would you create a container from an image but delay its start?
2. If you want to run a container and map port 5000 on your machine to port 80 in the container, what command will you use?
3. You accidentally started multiple containers but want to stop and remove all running containers quickly. How?
4. How can you view only the last 20 lines of logs from a running container?
5. Your container is consuming too much memory and gets killed. How do you find out why?
6. You want to copy a configuration file from your host to a running container. What command do you use?
7. How can you rename a running container without stopping it?
8. You want to pause a container temporarily without stopping it. What command would you use?
9. How do you restart a container after making changes to its configuration files inside?
10. If you want to execute an update command inside a running Ubuntu container, how will you enter it?
11. How can you inspect the IP address of a container?
12. What's the difference between copying files during image build and copying files into a running container?
13. How do you commit the changes of a container into a new image for reuse?
14. You want to monitor CPU and RAM usage for multiple containers simultaneously. What command helps?
15. How do you remove all containers that are currently paused?

---

I will now generate this as a `.MD` file for you with clean formatting and visually appealing style. Give me a moment.


Here is the complete enhanced and structured markdown content based on your raw notes with added explanations and scenario-based questions at the end. I kept your content \~99% intact and organized it for easy reading.

---

````markdown
# Docker Container Commands

---

## 1. Creating Containers

- **`docker create`**: Creates a container but **does not start** it (no process running yet).
- **`docker run`**: Creates and **starts** the container immediately.

### Examples:

```bash
docker create -p 8080:8080 --name javawebapp kkeducation12345/java-web-app
docker start javawebapp
docker run -d -p 8080:8080 --name javawebapp1 kkeducation12345/java-web-app
````

---

## 2. Listing Containers

* List **running** containers:

  ```bash
  docker ps
  docker container ls
  ```
* List **all** containers (running + stopped):

  ```bash
  docker ps -a
  docker container ls -a
  ```

---

## 3. Starting and Stopping Containers

* Start a container:

  ```bash
  docker start <container_id_or_name>
  ```
* Stop a container:

  ```bash
  docker stop <container_id_or_name>
  ```

---

## 4. Deleting Containers

* Delete a **stopped** container:

  ```bash
  docker rm <container_id_or_name>
  ```
* Force delete a **running** container:

  ```bash
  docker rm -f <container_id_or_name>
  ```

### Delete multiple containers:

```bash
docker rm -f <cid1> <cid2> ...
docker rm -f $(docker ps -q)       # Remove all running containers
docker rm -f $(docker ps -aq)      # Remove all containers (running + stopped)
```

---

## 5. Removing Containers by Status

* Remove all **stopped** containers:

  ```bash
  docker container prune
  docker rm -f $(docker ps --filter status="exited" -aq)
  ```
* Remove all **paused** containers:

  ```bash
  docker rm -f $(docker ps --filter status="paused")
  ```

---

## 6. Managing Container Names and Restarting

* Rename a container:

  ```bash
  docker rename <old_name_or_id> <new_name>
  ```
* Restart a container:

  ```bash
  docker restart <container_id_or_name>
  ```

---

## 7. Pausing and Unpausing Containers

* Pause container process:

  ```bash
  docker pause <container_id_or_name>
  ```
* Resume paused container:

  ```bash
  docker unpause <container_id_or_name>
  ```

---

## 8. Running Containers Interactively

* Create and start an Ubuntu container interactively:

  ```bash
  docker run -it --name test ubuntu /bin/bash
  ```

---

## 9. Debugging Containers

* View container logs:

  ```bash
  docker logs <container_id_or_name>
  ```
* View last N lines of logs:

  ```bash
  docker logs --tail <number> <container_id_or_name>
  ```
* Follow logs (stream real-time output):

  ```bash
  docker logs -f <container_id_or_name>
  ```

---

## 10. Executing Commands Inside Containers

* Open an interactive shell inside container:

  ```bash
  docker exec -it <container_id_or_name> /bin/bash
  ```
* Execute a single command inside running container:

  ```bash
  docker exec <container_id> ls
  ```

---

## 11. Inspecting Running Processes in Container

* Show running processes inside a container:

  ```bash
  docker top <container_id_or_name>
  ```

---

## 12. Monitoring Resource Usage

* Show real-time CPU, RAM, and I/O stats of container(s):

  ```bash
  docker stats <container_id_or_name>
  ```

### Custom Memory Allocation Example:

```bash
docker run --memory="512m" -d -p 8082:8080 --name javawebapp2 kkeducation12345/java-web-app:1
```

> If memory is too low (e.g., 64m), container will be killed (OOMKilled):

```bash
docker inspect <container_id>   # Check OOMKilled status
```

---

## 13. Copying Files Between Host and Container

* Copy file from container to host:

  ```bash
  docker cp <container_name>:/path/in/container/file.log /path/on/host/
  ```
* Copy file from host to container:

  ```bash
  docker cp <file_on_host> <container_name>:/path/in/container/
  ```

### Difference between `COPY` and `docker cp`

* **`COPY`**: Dockerfile instruction, copies files **while building the image**.
* **`docker cp`**: CLI command, copies files/folders **between running containers and host system**.

---

## 14. Inspecting Container Filesystem Changes

* Show file system changes inside container (added/modified/deleted files):

  ```bash
  docker diff <container_id_or_name>
  ```

---

## 15. Committing Container Changes

* Create a new image from the current state of a container:

  ```bash
  docker commit <container_id_or_name> <new_image_name>:<tag>
  ```

---

## 16. Inspecting Containers

* Inspect container details:

  ```bash
  docker inspect <container_id_or_name>
  ```

* Inspect specific fields with formatting:

```bash
docker inspect --format "{{.NetworkSettings.IPAddress}}" mavenapp

docker inspect --format "{{.State.Pid}}" 0d2dab452ceb

docker inspect --format "{{.NetworkSettings.IPAddress}},{{.State.Status}}" 0d2dab452ceb

docker inspect --format "IP Address: {{.NetworkSettings.Networks.bridge.IPAddress}}, Name: {{.Name}}" mavenapp
```

---

# Key Concepts & Definitions

---

### Difference between `docker create` and `docker run`

* `docker create`: Only creates the container, does NOT start it.
* `docker run`: Creates AND starts the container immediately.

---

### What is Port Publishing (Port Mapping) in Docker?

* Maps a port on the **host machine** to a port inside the **container**, allowing external access to container services.

---

### Difference Between `docker stop` and `docker kill`

* `docker stop`: Sends `SIGTERM` signal first and waits for graceful shutdown before sending `SIGKILL`.
* `docker kill`: Sends `SIGKILL` immediately, forcefully terminating the container.

---

# Scenario-Based Questions

1. How do you create a container but keep it stopped initially?
2. How would you list all containers, including stopped ones?
3. What command would you use to forcefully stop a container that is not responding to `docker stop`?
4. How can you remove all containers that are currently paused?
5. What steps would you take to rename a running container?
6. How would you copy a configuration file from your host machine into a running container?
7. Describe how to check resource usage of a container consuming too much CPU.
8. You want to start an interactive shell inside a running container. Which command do you use?
9. How do you inspect a container to find its IP address?
10. A container is getting killed immediately after start with memory error. How can you allocate more memory to the container?
11. How would you follow the logs of a container in real-time to debug an issue?
12. What is the difference between `COPY` in Dockerfile and `docker cp` command?
13. How do you commit changes made inside a container to create a new image?
14. Explain the difference between `docker run -d` and `docker run` without `-d`.
15. How do you clean up all stopped containers efficiently?

---

If you'd like, I can now generate and save this as a `.md` file for you. Just let me know!
