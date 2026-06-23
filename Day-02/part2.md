# Day 02 - Chapter 02: Podman Basics

**Date:** 2026-06-23  
**Course:** DO188 - Red Hat OpenShift Developer I: Introduction to Containers with Podman  
**Chapter Goal:** Manage and run containers with Podman.

## Daily Preparation Schedule

| Time | Task |
| --- | --- |
| 30 min | Read Chapter 2 concepts carefully |
| 45 min | Practice basic Podman commands |
| 60 min | Complete guided exercises |
| 45 min | Complete the chapter lab |
| 20 min | Take screenshots for LinkedIn |
| 20 min | Update GitHub notes and commands |
| 10 min | Write a short LinkedIn learning summary |

## Chapter 2 Objectives

By the end of this chapter, I should be able to:

- Run containerized services with Podman.
- Understand how containers communicate with each other.
- Expose container ports to the host machine.
- Inspect and access running containers.
- Start, stop, restart, pause, kill, and remove containers.
- Copy files into and out of containers.
- Use Podman networks for container-to-container communication.

## 1. Concept: Podman Basics

Podman is a container management tool used to run, manage, inspect, and troubleshoot containers locally. It is especially important in Red Hat environments because it supports OCI container standards and works well with rootless container workflows.

Unlike some container tools, Podman is daemonless. This means it does not require one always-running background service to manage containers. Podman talks directly to containers, images, and registries.

## 2. Definition

**Podman** is an open source command-line tool for managing containers and container images.

**Container image** is a packaged application environment that includes the application and its dependencies.

**Container** is a running or stopped instance created from a container image.

Simple relationship:

```text
Image -> used to create -> Container
```

Example:

```bash
podman pull registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Hello Red Hat"
```

## 3. Examples

### Check Podman Version

```bash
podman -v
```

### Pull an Image

```bash
podman pull registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5
```

### List Local Images

```bash
podman images
```

### Run a Simple Container

```bash
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Hello Red Hat"
```

### Run a Container with Environment Variables

```bash
podman run -e GREET=Hello -e NAME="Red Hat" \
  registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv GREET NAME
```

### Run a Web Server Container

```bash
podman run -d --name webserver -p 8080:8080 \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

### Access the Web Server

```bash
curl http://localhost:8080
```

## 4. Explanation

### Running Containers

The `podman run` command creates and starts a container from an image. If the image does not exist locally, Podman can pull it from a registry first.

Example:

```bash
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Learning Podman"
```

If the command inside the container finishes, the container stops. This is normal behavior.

### Listing Containers

List running containers:

```bash
podman ps
```

List all containers, including stopped containers:

```bash
podman ps --all
```

### Naming Containers

Container names make management easier.

```bash
podman run --name my-container \
  registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Named container"
```

### Detached Mode

Detached mode runs a container in the background.

```bash
podman run -d --name httpd -p 8080:8080 \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

### Port Forwarding

Container networking is isolated by default. Port forwarding allows access from the host machine.

Format:

```text
HOST_PORT:CONTAINER_PORT
```

Example:

```bash
podman run -d -p 8080:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
```

This maps port `8080` on my machine to port `8080` inside the container.

### Podman Networks

Podman networks allow containers to communicate with each other. A custom network is useful when multiple containers are part of the same application.

Create a network:

```bash
podman network create lab-net
```

List networks:

```bash
podman network ls
```

Inspect a network:

```bash
podman network inspect lab-net
```

Run a container on a network:

```bash
podman run -d --name app --net lab-net \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

When DNS is enabled on a custom network, containers can communicate by container name.

### Accessing Containers

Run a command inside a running container:

```bash
podman exec CONTAINER_NAME command
```

Example:

```bash
podman exec webserver ls /var/www/html
```

Open an interactive shell:

```bash
podman exec -it webserver /bin/bash
```

### Copying Files

Copy a file from host to container:

```bash
podman cp index.html webserver:/var/www/html/
```

Copy a file from container to host:

```bash
podman cp webserver:/etc/secret-file ./solution
```

### Inspecting Containers

Get full container details:

```bash
podman inspect webserver
```

Get only the container status:

```bash
podman inspect --format='{{.State.Status}}' webserver
```

Check whether it is running:

```bash
podman inspect --format='{{.State.Running}}' webserver
```

### Container Lifecycle Commands

Stop a container gracefully:

```bash
podman stop webserver
```

Stop a container immediately:

```bash
podman kill webserver
```

Restart a container:

```bash
podman restart webserver
```

Pause a container:

```bash
podman pause webserver
```

Unpause a container:

```bash
podman unpause webserver
```

Remove a stopped container:

```bash
podman rm webserver
```

Force remove a running container:

```bash
podman rm webserver --force
```

## 5. Labs

## Guided Exercise 1: Creating Containers with Podman

### Goal

Practice creating containers with `podman run`.

### Commands Practiced

```bash
lab start basics-creating
podman pull registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5
podman images
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Hello Red Hat"
podman ps
podman run -e GREET=Hello -e NAME="Red Hat" registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv GREET NAME
podman run -d -p 8080:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
podman ps
lab finish basics-creating
```

### Screenshot Ideas

- `podman images`
- `podman run ... echo "Hello Red Hat"`
- `podman ps`
- Browser or `curl http://localhost:8080`

## Guided Exercise 2: Accessing Containerized Network Services

### Goal

Practice exposing container services with port forwarding.

### Important Concept

The container has its own network namespace. To access a service from the host, I must publish a port.

Example:

```bash
podman run -d --name httpd -p 8080:8080 \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

Access from host:

```bash
curl http://localhost:8080
```

### Screenshot Ideas

- Running container with port mapping.
- Output of `podman ps`.
- Browser showing the web page.

## Guided Exercise 3: Accessing Containers

### Goal

Practice using `podman exec` and `podman cp`.

### Commands Practiced

```bash
podman exec CONTAINER_NAME ls /var/www/html
podman exec -it CONTAINER_NAME /bin/bash
podman cp index.html CONTAINER_NAME:/var/www/html/
podman cp CONTAINER_NAME:/path/file ./local-file
```

### Screenshot Ideas

- `podman exec` command output.
- File copied into a container.
- Web page updated after copying `index.html`.

## Guided Exercise 4: Managing the Container Lifecycle

### Goal

Practice inspecting, stopping, restarting, killing, and removing containers.

### Commands Practiced

```bash
lab start basics-lifecycle
podman run --name httpd -d -p 8080:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
podman ps
podman inspect --format='{{.State.Status}}' httpd
podman inspect --format='{{.State.Running}}' httpd
podman stop httpd
podman restart httpd
podman rm httpd --force
podman run --name greeter -d registry.ocp4.example.com:8443/redhattraining/podman-greeter-ignore-sigterm
podman stop greeter --time=5
podman restart greeter
podman kill greeter
lab finish basics-lifecycle
```

### Screenshot Ideas

- `podman inspect --format='{{.State.Status}}' httpd`
- `podman stop httpd`
- `podman restart httpd`
- `podman rm httpd --force`

## Chapter Lab: Podman Basics

### Goal

Use Podman to manage local containers, copy files, use networks, and expose a service.

### Lab Tasks

1. Start the lab:

```bash
lab start basics-podman
```

2. Verify the secret container is running:

```bash
podman ps --format='{{.Names}}'
```

3. Copy a file from the secret container:

```bash
podman cp basics-podman-secret:/etc/secret-file solution
```

4. Create a custom network:

```bash
podman network create lab-net
```

5. Start the server container:

```bash
podman run -d --name basics-podman-server \
  --net lab-net -p 8080:8080 \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

6. Copy the web page into the server container:

```bash
podman cp index.html basics-podman-server:/var/www/html/
```

7. Start the client container:

```bash
podman run -d --name basics-podman-client \
  --net lab-net \
  registry.ocp4.example.com:8443/ubi8/httpd-24
```

8. Confirm DNS communication between containers:

```bash
podman network inspect lab-net
podman exec basics-podman-client curl -s http://basics-podman-server:8080
```

9. Finish the lab:

```bash
lab finish basics-podman
```

## Key Commands Summary

| Command | Purpose |
| --- | --- |
| `podman pull IMAGE` | Download an image |
| `podman images` | List local images |
| `podman run IMAGE` | Create and start a container |
| `podman run -d IMAGE` | Run container in background |
| `podman run --name NAME IMAGE` | Assign a container name |
| `podman run -p HOST:CONTAINER IMAGE` | Publish a container port |
| `podman ps` | List running containers |
| `podman ps --all` | List all containers |
| `podman exec CONTAINER COMMAND` | Run command inside container |
| `podman cp SRC DEST` | Copy files between host and container |
| `podman inspect CONTAINER` | View detailed container info |
| `podman stop CONTAINER` | Stop gracefully |
| `podman kill CONTAINER` | Stop forcefully |
| `podman restart CONTAINER` | Restart container |
| `podman rm CONTAINER` | Remove stopped container |
| `podman network create NAME` | Create a network |
| `podman network inspect NAME` | Inspect network details |

## My LinkedIn Post Draft

Today was Day 02 of my DO188 exam preparation.

I studied Chapter 2: Podman Basics. The main focus was learning how to create and manage containers with Podman, expose containerized services with port forwarding, use custom Podman networks, access running containers, copy files into and out of containers, and manage the full container lifecycle.

Key topics I practiced:

- `podman run`
- `podman ps`
- `podman exec`
- `podman cp`
- `podman inspect`
- `podman stop`
- `podman restart`
- `podman rm`
- `podman network create`
- Container port forwarding
- Container-to-container communication by DNS name

The most useful part was understanding how a server container and client container can communicate over a custom Podman network, and how port forwarding makes a containerized service accessible from the host machine.

I also completed the Chapter 2 lab, where I practiced running multiple containers, copying files, creating a custom network, and verifying communication between containers.

#RedHat #Podman #Containers #OpenShift #Linux #DevOps #DO188 #LearningInPublic

## GitHub Commit Message

```text
Add Day 02 notes for Podman basics
```

## Personal Reflection

Today I learned that Podman is not only used to start containers. It also provides a complete workflow for local container management, including networking, port forwarding, file copying, inspection, and lifecycle operations. This chapter is very important for the EX188 exam because these are the practical commands I will use again and again.

# Practice Lab Bank - Chapter 2

Use these labs for hands-on practice before moving to Chapter 3. Each topic has basic labs, scenario-based labs, intermediate labs, and an industry/interview-style question with solution.

## Topic 1: Images and Basic Container Execution

### Lab 1.1 - Basic: Check Podman and Pull an Image

**Question:** Check the Podman version, pull the UBI minimal image, and list local images.

**Solution:**

```bash
podman -v
podman pull registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5
podman images
```

**Expected result:** The image appears in the local image list.

### Lab 1.2 - Basic: Run a One-Time Command

**Question:** Run a container that prints `Hello from Podman`.

**Solution:**

```bash
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Hello from Podman"
```

**Expected result:** The terminal prints `Hello from Podman`.

### Lab 1.3 - Scenario: Verify Container Stops After Command

**Question:** Run a container that prints a message, then prove that it is not running anymore.

**Solution:**

```bash
podman run registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Task completed"
podman ps
podman ps --all
```

**Expected result:** `podman ps` does not show the container, but `podman ps --all` shows it as exited.

### Lab 1.4 - Intermediate: Run and Auto Remove a Container

**Question:** Run a short-lived container and automatically remove it after execution.

**Solution:**

```bash
podman run --rm registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Temporary container"
podman ps --all
```

**Expected result:** The container should not remain in `podman ps --all`.

### Lab 1.5 - Industry/Interview Question

**Question:** What is the difference between an image and a container?

**Answer:** An image is a packaged application environment with dependencies. A container is a running or stopped instance created from that image. Images can exist without containers, but containers need images.

## Topic 2: Naming Containers and Listing Containers

### Lab 2.1 - Basic: Create a Named Container

**Question:** Create a container named `day02-test` that prints `Named container`.

**Solution:**

```bash
podman run --name day02-test registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 echo "Named container"
podman ps --all
```

**Expected result:** The container name `day02-test` appears in the output.

### Lab 2.2 - Basic: List Only Container Names

**Question:** Display only container names.

**Solution:**

```bash
podman ps --all --format='{{.Names}}'
```

**Expected result:** Only names are displayed.

### Lab 2.3 - Scenario: Find a Stopped Container

**Question:** A developer says their container disappeared after running a command. Show that the container still exists but is stopped.

**Solution:**

```bash
podman ps
podman ps --all
```

**Expected result:** Running containers appear with `podman ps`; stopped containers appear only with `podman ps --all`.

### Lab 2.4 - Intermediate: Show Container ID, Status, and Name

**Question:** List all containers with only ID, status, and name.

**Solution:**

```bash
podman ps --all --format='{{.ID}} {{.Status}} {{.Names}}'
```

**Expected result:** Output is clean and easy to read.

### Lab 2.5 - Industry/Interview Question

**Question:** Why should you assign names to containers?

**Answer:** Names make containers easier to manage. It is easier to run `podman stop webserver` than to copy a long container ID.

## Topic 3: Environment Variables

### Lab 3.1 - Basic: Pass One Environment Variable

**Question:** Pass `NAME=RedHat` into a container and print it.

**Solution:**

```bash
podman run -e NAME=RedHat registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv NAME
```

**Expected result:** Output shows `RedHat`.

### Lab 3.2 - Basic: Pass Multiple Environment Variables

**Question:** Pass `APP=web` and `ENV=dev` into a container and print both.

**Solution:**

```bash
podman run -e APP=web -e ENV=dev registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv APP ENV
```

**Expected result:** Output shows both values.

### Lab 3.3 - Scenario: Configure App Environment

**Question:** Simulate a containerized app that receives database settings from environment variables.

**Solution:**

```bash
podman run --rm \
  -e DB_HOST=db.example.local \
  -e DB_USER=appuser \
  -e DB_NAME=appdb \
  registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv DB_HOST DB_USER DB_NAME
```

**Expected result:** The database configuration values print successfully.

### Lab 3.4 - Intermediate: Inspect Environment Variables

**Question:** Start a named container with an environment variable and inspect its configuration.

**Solution:**

```bash
podman run --name env-test -e COURSE=DO188 registry.ocp4.example.com:8443/ubi8/ubi-minimal:8.5 printenv COURSE
podman inspect env-test
```

**Expected result:** `COURSE=DO188` is visible in the inspect output.

### Lab 3.5 - Industry/Interview Question

**Question:** Why are environment variables useful in containers?

**Answer:** They allow configuration to change without modifying application code or rebuilding the image. For example, the same image can use different database hosts in development, testing, and production.

## Topic 4: Port Forwarding and Network Services

### Lab 4.1 - Basic: Run an HTTP Server

**Question:** Start an Apache HTTP server container and expose it on port `8080`.

**Solution:**

```bash
podman run -d --name web-basic -p 8080:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
podman ps
```

**Expected result:** `podman ps` shows `0.0.0.0:8080->8080/tcp`.

### Lab 4.2 - Basic: Test the Published Port

**Question:** Test the web server from the host.

**Solution:**

```bash
curl http://localhost:8080
```

**Expected result:** The server responds.

### Lab 4.3 - Scenario: Run Web Server on a Different Host Port

**Question:** Port `8080` is already used. Run the container on host port `8081`.

**Solution:**

```bash
podman run -d --name web-8081 -p 8081:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
curl http://localhost:8081
```

**Expected result:** The service is accessible from `localhost:8081`.

### Lab 4.4 - Intermediate: Bind to Localhost Only

**Question:** Publish the container only on `127.0.0.1`.

**Solution:**

```bash
podman run -d --name web-local -p 127.0.0.1:8082:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
podman ps
curl http://127.0.0.1:8082
```

**Expected result:** The service is bound to localhost.

### Lab 4.5 - Industry/Interview Question

**Question:** What does `-p 8080:8080` mean?

**Answer:** The first `8080` is the host port. The second `8080` is the container port. Traffic to the host on port `8080` is forwarded to port `8080` inside the container.

## Topic 5: Podman Networks and DNS

### Lab 5.1 - Basic: Create a Network

**Question:** Create a network named `day02-net`.

**Solution:**

```bash
podman network create day02-net
podman network ls
```

**Expected result:** `day02-net` appears in the network list.

### Lab 5.2 - Basic: Inspect DNS Setting

**Question:** Inspect the network and verify whether DNS is enabled.

**Solution:**

```bash
podman network inspect day02-net
```

**Expected result:** The inspect output includes network details such as DNS configuration.

### Lab 5.3 - Scenario: Connect Two Containers by Name

**Question:** Start a server and client container on the same network. Use the client to access the server by name.

**Solution:**

```bash
podman run -d --name net-server --net day02-net registry.ocp4.example.com:8443/ubi8/httpd-24
podman run -d --name net-client --net day02-net registry.ocp4.example.com:8443/ubi8/httpd-24
podman exec net-client curl -s http://net-server:8080
```

**Expected result:** The client reaches the server by DNS name.

### Lab 5.4 - Intermediate: Attach a Running Container to a Network

**Question:** Start a container without the custom network, then connect it to `day02-net`.

**Solution:**

```bash
podman run -d --name late-client registry.ocp4.example.com:8443/ubi8/httpd-24
podman network connect day02-net late-client
podman exec late-client curl -s http://net-server:8080
```

**Expected result:** After connecting to the network, the container can reach `net-server`.

### Lab 5.5 - Industry/Interview Question

**Question:** Why should you use a custom Podman network instead of only the default network?

**Answer:** Custom networks provide better isolation and DNS-based container discovery. Containers on the same custom network can communicate by container name.

## Topic 6: Accessing Containers with `podman exec`

### Lab 6.1 - Basic: Run a Command Inside a Container

**Question:** Run `hostname` inside a running container.

**Solution:**

```bash
podman run -d --name exec-test registry.ocp4.example.com:8443/ubi8/httpd-24
podman exec exec-test hostname
```

**Expected result:** The container hostname is displayed.

### Lab 6.2 - Basic: List Web Directory

**Question:** List `/var/www/html` inside the web container.

**Solution:**

```bash
podman exec exec-test ls /var/www/html
```

**Expected result:** Directory contents are displayed.

### Lab 6.3 - Scenario: Troubleshoot a Running Web Container

**Question:** Check whether the Apache process is running inside the container.

**Solution:**

```bash
podman exec exec-test ps -ef
```

**Expected result:** The HTTP server process is visible.

### Lab 6.4 - Intermediate: Open an Interactive Shell

**Question:** Open an interactive shell inside a running container.

**Solution:**

```bash
podman exec -it exec-test /bin/bash
```

Inside the container:

```bash
whoami
pwd
exit
```

**Expected result:** You can interact with the container shell.

### Lab 6.5 - Industry/Interview Question

**Question:** What is the difference between `podman run` and `podman exec`?

**Answer:** `podman run` creates and starts a new container. `podman exec` runs a command inside an already running container.

## Topic 7: Copying Files with `podman cp`

### Lab 7.1 - Basic: Copy File from Host to Container

**Question:** Create an `index.html` file and copy it into a web container.

**Solution:**

```bash
echo "<h1>Day 02 Podman Practice</h1>" > index.html
podman cp index.html exec-test:/var/www/html/
curl http://localhost:8080
```

**Expected result:** The web page shows the new HTML content if the container port is published.

### Lab 7.2 - Basic: Copy File from Container to Host

**Question:** Copy `/etc/os-release` from a container to the host.

**Solution:**

```bash
podman cp exec-test:/etc/os-release ./os-release-copy
cat os-release-copy
```

**Expected result:** The host now has a copy of the file.

### Lab 7.3 - Scenario: Replace Web Content

**Question:** A developer gives you a new `index.html`. Replace the file inside the running web container.

**Solution:**

```bash
echo "<h1>Updated by podman cp</h1>" > index.html
podman cp index.html exec-test:/var/www/html/
curl http://localhost:8080
```

**Expected result:** The web content is updated.

### Lab 7.4 - Intermediate: Verify File Copy with `podman exec`

**Question:** Copy a file into a container and verify it from inside the container.

**Solution:**

```bash
echo "verification file" > verify.txt
podman cp verify.txt exec-test:/tmp/verify.txt
podman exec exec-test cat /tmp/verify.txt
```

**Expected result:** The container prints `verification file`.

### Lab 7.5 - Industry/Interview Question

**Question:** When would you use `podman cp`?

**Answer:** Use it to copy files into or out of a container for testing, debugging, collecting logs, replacing test content, or retrieving generated files.

## Topic 8: Inspecting Containers

### Lab 8.1 - Basic: Inspect a Container

**Question:** Inspect a running container.

**Solution:**

```bash
podman inspect exec-test
```

**Expected result:** Podman prints detailed JSON information.

### Lab 8.2 - Basic: Get Container Status

**Question:** Print only the status of the container.

**Solution:**

```bash
podman inspect --format='{{.State.Status}}' exec-test
```

**Expected result:** Output shows `running`, `exited`, or another status.

### Lab 8.3 - Scenario: Verify if App Is Running

**Question:** A web app is not responding. Check whether its container is running.

**Solution:**

```bash
podman inspect --format='{{.State.Running}}' exec-test
podman ps
```

**Expected result:** `true` means the container is running; `false` means it is stopped.

### Lab 8.4 - Intermediate: Inspect Network Settings

**Question:** Show the network configuration of a container.

**Solution:**

```bash
podman inspect exec-test
```

Look for network-related fields in the JSON output.

**Expected result:** You can identify network, IP, and port-related configuration.

### Lab 8.5 - Industry/Interview Question

**Question:** Why is `podman inspect` useful?

**Answer:** It gives detailed container metadata, including state, image, command, environment variables, networking, mounts, and process information. It is one of the most important troubleshooting commands.

## Topic 9: Container Lifecycle Management

### Lab 9.1 - Basic: Stop a Container

**Question:** Stop a running container gracefully.

**Solution:**

```bash
podman stop exec-test
podman ps --all
```

**Expected result:** The container status becomes exited.

### Lab 9.2 - Basic: Restart a Container

**Question:** Restart the stopped container.

**Solution:**

```bash
podman restart exec-test
podman ps
```

**Expected result:** The container is running again.

### Lab 9.3 - Scenario: Remove a Container

**Question:** Remove a container after stopping it.

**Solution:**

```bash
podman stop exec-test
podman rm exec-test
podman ps --all
```

**Expected result:** The container no longer appears.

### Lab 9.4 - Intermediate: Force Remove a Running Container

**Question:** Start a container and remove it while it is still running.

**Solution:**

```bash
podman run -d --name force-test registry.ocp4.example.com:8443/ubi8/httpd-24
podman rm force-test --force
podman ps --all
```

**Expected result:** The container is removed.

### Lab 9.5 - Industry/Interview Question

**Question:** What is the difference between `podman stop`, `podman kill`, and `podman rm`?

**Answer:** `podman stop` gracefully stops a container by sending SIGTERM first. `podman kill` forcefully stops it by sending SIGKILL. `podman rm` removes a stopped container from the system.

## Topic 10: Complete Chapter 2 Scenario Labs

### Scenario Lab 10.1 - Mini Web Deployment

**Question:** Deploy a web container named `portfolio-web`, expose it on port `8085`, copy a custom `index.html`, and test it.

**Solution:**

```bash
podman run -d --name portfolio-web -p 8085:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
echo "<h1>My DO188 Portfolio</h1>" > index.html
podman cp index.html portfolio-web:/var/www/html/
curl http://localhost:8085
```

### Scenario Lab 10.2 - Client and Server Network

**Question:** Create a custom network and prove that a client container can reach a server container by DNS name.

**Solution:**

```bash
podman network create app-net
podman run -d --name app-server --net app-net registry.ocp4.example.com:8443/ubi8/httpd-24
podman run -d --name app-client --net app-net registry.ocp4.example.com:8443/ubi8/httpd-24
podman exec app-client curl -s http://app-server:8080
```

### Scenario Lab 10.3 - Container Health Check Practice

**Question:** Start a web container, verify status, verify running state, and test the port.

**Solution:**

```bash
podman run -d --name health-web -p 8090:8080 registry.ocp4.example.com:8443/ubi8/httpd-24
podman inspect --format='{{.State.Status}}' health-web
podman inspect --format='{{.State.Running}}' health-web
curl http://localhost:8090
```

### Scenario Lab 10.4 - File Recovery from Container

**Question:** Copy a file from inside a container to the host for troubleshooting evidence.

**Solution:**

```bash
podman run -d --name evidence-web registry.ocp4.example.com:8443/ubi8/httpd-24
podman cp evidence-web:/etc/os-release ./evidence-os-release
cat evidence-os-release
```

### Scenario Lab 10.5 - Cleanup Practice

**Question:** Stop and remove all practice containers from this chapter.

**Solution:**

```bash
podman ps --all
podman stop --all
podman rm --all
podman ps --all
```

**Note:** Use this only after you finish practice work.

## Interview Questions - Chapter 2

### Question 1

**What is Podman?**

**Answer:** Podman is an open source tool used to manage OCI containers and images. It can pull images, run containers, inspect containers, manage networks, and handle the container lifecycle.

### Question 2

**Why is Podman called daemonless?**

**Answer:** Podman does not require a continuously running background daemon to manage containers. It interacts directly with containers, images, and registries.

### Question 3

**How do you run a container in detached mode?**

**Answer:**

```bash
podman run -d IMAGE
```

### Question 4

**How do you expose a containerized web application to the host?**

**Answer:**

```bash
podman run -d -p 8080:8080 IMAGE
```

### Question 5

**How do containers communicate by name in Podman?**

**Answer:** Create a custom Podman network and attach containers to it. Containers on that network can communicate by container name when DNS is enabled.

### Question 6

**How do you copy a file into a container?**

**Answer:**

```bash
podman cp local-file CONTAINER:/path/
```

### Question 7

**How do you run a command inside a running container?**

**Answer:**

```bash
podman exec CONTAINER command
```

### Question 8

**How do you check the status of a container with Go template output?**

**Answer:**

```bash
podman inspect --format='{{.State.Status}}' CONTAINER
```

### Question 9

**Why might `podman rm CONTAINER` fail?**

**Answer:** It fails if the container is running. Stop it first with `podman stop`, or force remove it with `podman rm --force`.

### Question 10

**What commands should you practice the most for Chapter 2?**

**Answer:** Practice `podman run`, `podman ps`, `podman exec`, `podman cp`, `podman inspect`, `podman stop`, `podman restart`, `podman rm`, and `podman network`.
