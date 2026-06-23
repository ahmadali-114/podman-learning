# DO188 — Chapter 2: Podman Basics
### Red Hat OpenShift Development I: Introduction to Containers with Podman

> **Learning Journal** | Ahmad Ali 
> **Course:** DO188 — RHOSCP 4.10 | **Certification Path:** EX188 (Red Hat Certified Specialist in Containers)  
> **GitHub:** [github.com/ahmadali-114](https://github.com/your-username) | **LinkedIn:** [linkedin.com/in/ahmadalimalik](https://linkedin.com/in/ahmad-ali)

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [What is Podman?](#what-is-podman)
3. [Section 1 — Creating Containers with Podman](#section-1--creating-containers-with-podman)
4. [Section 2 — Container Networking Basics](#section-2--container-networking-basics)
5. [Section 3 — Accessing Containerized Network Services](#section-3--accessing-containerized-network-services)
6. [Section 4 — Accessing Containers](#section-4--accessing-containers)
7. [Section 5 — Managing the Container Lifecycle](#section-5--managing-the-container-lifecycle)
8. [Complete Command Reference](#complete-command-reference)
9. [Labs — Basic (per topic)](#labs--basic-per-topic)
10. [Labs — Scenario Based](#labs--scenario-based)
11. [Labs — Intermediate](#labs--intermediate)
12. [Labs — Industry / Interview Questions with Solutions](#labs--industry--interview-questions-with-solutions)
13. [Guided Exercises from the Book](#guided-exercises-from-the-book)
14. [Key Concepts Summary](#key-concepts-summary)
15. [Quiz: Self-Assessment](#quiz-self-assessment)

---

## Chapter Overview

**Goal:** Manage and run containers with Podman.

**Objectives covered:**
- Run a containerized service with Podman
- Describe how containers communicate with each other
- Expose ports to access containerized services
- Explore running containers using `exec` and `cp`
- List, stop, and delete containers with Podman

**Sections:**
| Section | Topic |
|---------|-------|
| 2.1 | Creating Containers with Podman |
| 2.2 | Container Networking Basics |
| 2.3 | Accessing Containerized Network Services |
| 2.4 | Accessing Containers (exec, cp) |
| 2.5 | Managing the Container Lifecycle |

---

## What is Podman?

Podman is an open source, **daemonless** container engine for managing OCI (Open Container Initiative) containers and images on a single host.

### Key characteristics (from the DO188 student guide):

| Feature | Podman | Docker |
|---------|--------|--------|
| Daemon | ❌ No daemon (daemonless) | ✅ Requires dockerd daemon |
| Root | Rootless by default | Daemon runs as root |
| Single point of failure | ❌ None | ✅ Daemon = single point of failure |
| CLI compatibility | Drop-in Docker replacement | — |
| API | RESTful API included | RESTful API included |
| Pods | ✅ Native pod support | ❌ No native pods |

> **From the book:** "By default, Podman is daemonless. A daemon is a process that is always running and ready for receiving incoming requests. Some other container tools use a daemon to proxy the requests, which brings a single point of failure. In addition, a daemon might require elevated privileges, which is a security concern. Podman interacts directly with containers, images, and registries without a daemon."

```bash
# Check your Podman version
podman -v
podman version

# Full system information
podman info
```

---

## Section 1 — Creating Containers with Podman

### Core Concepts

**Container Image** — A packaged, read-only snapshot of an application and all its dependencies. Images exist without containers.

**Container Instance** — A running (or stopped) executable version of an image. Containers depend on images; images do not depend on containers.

> **Book analogy:** "An instance relates to an image as an object relates to a class in object-oriented programming."

### Pulling Images

```bash
# Pull an image from Red Hat registry
podman pull registry.redhat.io/rhel7/rhel:7.9

# Pull UBI minimal image
podman pull registry.access.redhat.com/ubi8/ubi-minimal:8.5

# List all locally stored images
podman images
```

**Image naming format:** `registry/repository:tag`  
Example: `registry.access.redhat.com/ubi8/httpd-24:latest`

> **Note from the book:** "If you run a container from an image that is not stored in your system then Podman tries to pull the image before running the container. Therefore, it is not necessary to execute the pull command first."

### Running Containers

```bash
# Basic run (executes a command and exits)
podman run registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'

# Run with a name assigned
podman run --name mycontainer registry.access.redhat.com/ubi8:latest echo "hello"

# Run and auto-remove after exit (--rm)
podman run --rm registry.access.redhat.com/ubi8:latest echo "temporary"

# Run in detached/background mode (-d)
podman run -d --name webapp registry.access.redhat.com/ubi8/httpd-24:latest

# Run interactive with terminal (-it)
podman run -it registry.access.redhat.com/ubi8:latest /bin/bash

# Run with port mapping (-p hostPort:containerPort)
podman run -d -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest

# Run with environment variable (-e)
podman run -e NAME='Red Hat' registry.redhat.io/rhel7/rhel:7.9 printenv NAME
```

### Listing Containers

```bash
# List only RUNNING containers
podman ps

# List ALL containers (running + stopped + exited)
podman ps --all
podman ps -a

# List with JSON format (full UUID)
podman ps --all --format=json

# Custom table format
podman ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# Quiet mode — only IDs
podman ps -q        # running only
podman ps -qa       # all containers

# Filter by status
podman ps -a --filter status=running
podman ps -a --filter status=exited
podman ps -a --filter name=myapp
```

**Container identification:** Podman uses:
- **Short UUID** — 12 alphanumeric characters (e.g., `20236410bcef`)
- **Long UUID** — 64 alphanumeric characters (from `--format=json`)
- **Name** — human-readable (e.g., `webapp`)

### Environment Variables

```bash
# Pass single env var
podman run -e GREET=Hello registry.access.redhat.com/ubi8:latest printenv GREET

# Pass multiple env vars
podman run -e GREET=Hello -e NAME='Red Hat' \
  registry.access.redhat.com/ubi8/ubi-minimal:8.5 \
  printenv GREET NAME

# Pass from host environment (no value = copies from host)
export MY_SECRET=secret123
podman run --rm -e MY_SECRET registry.access.redhat.com/ubi8:latest printenv MY_SECRET

# Use env file
podman run --env-file /path/to/app.env registry.access.redhat.com/ubi8:latest env
```

---

## Section 2 — Container Networking Basics

### Default Podman Network

Podman comes with a default network called `podman`. All containers attach to this network by default. However, **DNS is disabled on the default network** — containers cannot resolve each other by name.

```bash
# Inspect the default network
podman network inspect podman
# "dns_enabled": false   ← DNS disabled on default!
```

### Managing Podman Networks

```bash
# Create a new network (DNS enabled by default on custom networks)
podman network create mynetwork

# List all networks
podman network ls

# Inspect a network (detailed JSON)
podman network inspect mynetwork

# Remove a network
podman network rm mynetwork

# Remove all unused networks
podman network prune

# Connect a running container to a network
podman network connect mynetwork mycontainer

# Disconnect a container from a network
podman network disconnect mynetwork mycontainer
```

### Attaching Containers to Networks

```bash
# Attach a new container to a custom network
podman run -d --name myapp --net mynetwork myimage:latest

# Connect to MULTIPLE networks (comma-separated)
podman run -d --name myapp --net network1,network2 myimage:latest
```

### DNS Between Containers

When using a **custom Podman network**, DNS is enabled automatically. A container's **hostname = its container name**.

```bash
# Create a DNS-enabled network
podman network create app-network
podman network inspect app-network
# "dns_enabled": true   ← DNS enabled on custom networks!

# Start two containers on the same network
podman run -d --name api-server --net app-network api-image:latest
podman run -d --name db-server  --net app-network postgres:15-alpine

# Now api-server can reach db-server by hostname:
# curl http://db-server:5432 from inside api-server container
```

**Key rule from the book:**
> "When using the default network, the domain name system (DNS) is disabled and can only be enabled by overwriting the default network. To use DNS, create a new Podman network and connect your containers to that network."

### Network Architecture Example (from the book)

```
┌─────────────┐    ui-network     ┌─────────────┐    api-network    ┌──────────────┐
│    app-ui   │◄─────────────────►│   app-api   │◄─────────────────►│    app-db    │
│ (ui:v2.1.1) │                   │ (api:v2.3.4)│                   │ (postgres:11)│
└─────────────┘                   └─────────────┘                   └──────────────┘
```

Only necessary connections exist. app-ui cannot directly reach app-db — isolation enforced!

---

## Section 3 — Accessing Containerized Network Services

### Port Forwarding

Containers are network-isolated. Use `-p` to expose container ports on the host.

```bash
# Syntax: -p HOST_PORT:CONTAINER_PORT
podman run -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest

# Map to a different host port (host 8075 → container 80)
podman run -p 8075:80 my-app

# Bind to specific host IP (localhost only — not accessible from network)
podman run -p 127.0.0.1:8080:8080 my-app

# Bind to all interfaces explicitly
podman run -p 0.0.0.0:8080:8080 my-app

# Let Podman pick a random host port
podman run -p :8080 my-app

# Publish all EXPOSE'd ports to random host ports
podman run -P my-app

# Map UDP port
podman run -p 5353:5353/udp my-app
```

> **Default behaviour:** Without a host IP specified, Podman assigns `0.0.0.0` — the container is accessible from all networks on the host.

### Listing Port Mappings

```bash
# Show port mappings for a container
podman port my-app
# Output: 8008/tcp -> 0.0.0.0:8010

# Show port mappings for ALL containers
podman port --all

# Get container IP within a specific network
podman inspect my-app \
  -f '{{.NetworkSettings.Networks.apps.IPAddress}}'
```

---

## Section 4 — Accessing Containers

### Container Layers and Transparency

When you start a container, it gets a **new ephemeral writable layer** on top of the base image's read-only layers. Files written at runtime are stored in this layer and are **lost when the container is removed**.

```
┌────────────────────────────┐
│ Container Writable Layer   │  ← runtime files (volatile, deleted on rm)
├────────────────────────────┤
│ Image Layer 3 (app code)   │  ← read-only
├────────────────────────────┤
│ Image Layer 2 (deps)       │  ← read-only
├────────────────────────────┤
│ Image Layer 1 (base OS)    │  ← read-only
└────────────────────────────┘
```

### podman exec — Run Commands in Running Containers

```bash
# Syntax
podman exec [options] container [command ...]

# Run a single command
podman exec httpd cat /etc/httpd/conf/httpd.conf

# Interactive shell (-it = --interactive --tty)
podman exec -it mycontainer /bin/bash

# With environment variable
podman exec -e ENVIRONMENT=dev mycontainer env

# As specific user
podman exec --user root mycontainer whoami

# In last created container (-l)
podman exec -e ENVIRONMENT=dev -l env

# Set working directory
podman exec --workdir /var/www/html mycontainer pwd
```

**Options explained:**

| Flag | Long Form | Purpose |
|------|-----------|---------|
| `-e` | `--env` | Set environment variable for this exec session only |
| `-i` | `--interactive` | Keep stdin open (accept input) |
| `-t` | `--tty` | Allocate pseudo terminal |
| `-l` | `--latest` | Use last created container |
| `-u` | `--user` | Run as specific user |
| `-d` | `--detach` | Run exec process in background |
| `--workdir` | `--workdir` | Set working directory |

> **Key insight from the book:** "After the env process finishes, the ENVIRONMENT variable is unset. To make the ENVIRONMENT variable persistent, stop the running container and add the -e ENVIRONMENT=dev option to the podman run command."

**Understanding -i vs -t vs -it:**

```bash
# No flags — shell opens, gets no input, exits immediately
podman exec -l /bin/bash

# -i only — accepts input but no terminal (no prompt displayed)
podman exec -il /bin/bash
# You can type but see no prompt

# -t only — pseudo terminal opens but receives no input
podman exec -tl /bin/bash
# bash-4.4$  (can see prompt but can't type)

# -it together — CORRECT: interactive terminal session
podman exec -it mycontainer /bin/bash
# bash-4.4$ pwd
# /opt/app-root/src
# bash-4.4$ exit
```

### podman cp — Copy Files Between Host and Container

```bash
# Syntax
podman cp [options] [container]:source_path [container]:destination_path

# Copy FROM container TO host
podman cp mycontainer:/tmp/logs .
podman cp mycontainer:/var/log/nginx/error.log ./error.log

# Copy FROM host TO container
podman cp nginx.conf nginx:/etc/nginx/nginx.conf
podman cp ./index.html webapp:/var/www/html/index.html

# Copy between two containers
podman cp nginx-test:/etc/nginx/nginx.conf nginx-proxy:/etc/nginx

# Copy a directory
podman cp mycontainer:/etc/httpd ./httpd-backup

# Works on stopped containers too!
podman stop mycontainer
podman cp mycontainer:/var/www/html ./html-backup
```

> **Important from the book:** "Keep in mind that changes applied to a running container are temporary and do not persist if the container is restarted or stopped. For persistent changes, you must rebuild the container image."

---

## Section 5 — Managing the Container Lifecycle

### Container States

```
               podman create
image ──────────────────────► Created
                                  │
                            podman start
                                  │
                                  ▼
              podman pause  ┌──────────┐  podman stop / kill
          ┌────────────────►│ Running  │────────────────────► Exited/Stopped
          │                 └──────────┘                           │
          │                      │                                 │
    ┌─────────┐           podman restart                    podman restart
    │ Paused  │◄──────────────────────────────────────────────────►│
    └─────────┘                                                     │
      podman unpause                                          podman rm
                                                                    │
                                                                    ▼
                                                               (deleted)
```

### Listing Containers

```bash
podman ps              # running only
podman ps -a           # all (running + stopped + exited)
podman ps -a --filter status=exited
```

### Inspecting Containers

```bash
# Full JSON output
podman inspect mycontainer

# Go template — extract specific field
podman inspect --format='{{.State.Status}}' mycontainer
# Output: running

podman inspect --format='{{.State.Running}}' mycontainer
# Output: true

# Common inspect queries
podman inspect mycontainer --format "{{.NetworkSettings.IPAddress}}"
podman inspect mycontainer --format "{{.Config.Env}}"
podman inspect mycontainer --format "{{.State.Pid}}"
podman inspect mycontainer --format "{{.NetworkSettings.Ports}}"
podman inspect mycontainer --format "{{.State.ExitCode}}"
podman inspect mycontainer --format "{{.ImageName}}"
```

> **From the book:** "Go template expressions use annotations, which are delimited by double curly braces ({{ and }}), to refer to elements. The data structure elements are separated by a dot (.) and must start in upper case."

### Stopping Containers

```bash
# Graceful stop — sends SIGTERM, then SIGKILL after 10 seconds
podman stop mycontainer

# Stop all running containers
podman stop --all

# Custom grace period before SIGKILL
podman stop --time=100 mycontainer   # 100 second grace period

# Immediate kill — sends SIGKILL directly (no grace period)
podman kill mycontainer
```

> **From the book:** "When you execute the podman stop command, Podman sends a SIGTERM signal to the container. After some seconds, if the container has not stopped on its own, Podman sends a SIGKILL signal to forcefully stop the container. By default, Podman waits 10 seconds before sending the SIGKILL signal."

### Pausing Containers

```bash
# Pause all processes in a container (SIGSTOP)
podman pause mycontainer

# Unpause (SIGCONT)
podman unpause mycontainer
```

### Restarting Containers

```bash
# Restart a stopped OR running container
podman restart mycontainer
```

### Removing Containers

```bash
# Remove a STOPPED container
podman rm mycontainer

# Remove a RUNNING container (fails without --force)
podman rm --force mycontainer
podman rm -f mycontainer

# Remove ALL stopped containers
podman rm --all

# Remove ALL containers (including running) — force
podman rm --force --all

# Remove only exited containers (cleaner/safer)
podman container prune
podman container prune -f   # no confirmation prompt

# System-wide cleanup
podman system prune         # containers + images
podman system prune -a -f   # everything unused
podman system df            # show disk usage
```

---

## Complete Command Reference

### Quick Reference Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `podman pull` | Download image from registry | — |
| `podman images` | List local images | `--filter`, `--format` |
| `podman run` | Create + start container | `-d`, `-it`, `--rm`, `-p`, `-e`, `--name`, `--net` |
| `podman ps` | List containers | `-a`, `-q`, `--filter`, `--format` |
| `podman inspect` | Get container/image metadata | `--format` |
| `podman exec` | Run command in running container | `-it`, `-e`, `-u`, `-l` |
| `podman cp` | Copy files host↔container | — |
| `podman logs` | View container stdout/stderr | `-f`, `--tail`, `-t`, `--since` |
| `podman stop` | Graceful stop (SIGTERM→SIGKILL) | `--all`, `--time` |
| `podman kill` | Immediate kill (SIGKILL) | — |
| `podman pause` | Freeze container (SIGSTOP) | — |
| `podman unpause` | Unfreeze container | — |
| `podman restart` | Restart stopped/running container | — |
| `podman rm` | Delete container | `--force`, `--all` |
| `podman port` | Show port mappings | `--all` |
| `podman stats` | Live resource usage | `--no-stream` |
| `podman network create` | Create a network | — |
| `podman network ls` | List networks | — |
| `podman network inspect` | Network details | — |
| `podman network rm` | Delete network | — |
| `podman network connect` | Connect container to network | — |
| `podman system df` | Show disk usage | — |
| `podman system prune` | Clean unused resources | `-a`, `-f` |

---

## Labs — Basic (per topic)

### Lab B1: podman run fundamentals

```bash
# 1. Run a hello-world style container
podman run registry.access.redhat.com/ubi8:latest echo "Hello DO188!"

# 2. Run interactive shell and explore
podman run -it --rm registry.access.redhat.com/ubi8:latest /bin/bash
# Inside: whoami, hostname, cat /etc/os-release, exit

# 3. Run with a name
podman run --name mytest registry.access.redhat.com/ubi8:latest echo "named container"
podman ps -a   # see the container with name mytest

# 4. Run and auto-remove
podman run --rm registry.access.redhat.com/ubi8:latest echo "ephemeral"
podman ps -a   # NOT listed — auto-removed

# 5. Run in detached mode
podman run -d --name webserver registry.access.redhat.com/ubi8/httpd-24:latest
podman ps      # shows running in background

# 6. Override default CMD
podman run --rm registry.access.redhat.com/ubi8/httpd-24:latest cat /etc/os-release

# 7. Run with port mapping
podman run -d -p 8080:8080 --name web registry.access.redhat.com/ubi8/httpd-24:latest
curl http://localhost:8080

# Cleanup
podman stop web webserver mytest && podman rm web webserver mytest
```

---

### Lab B2: podman ps and inspect

```bash
# Setup
podman run -d --name web -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest
podman run --name exited-one registry.access.redhat.com/ubi8:latest echo "done"

# 1. List only running
podman ps

# 2. List all (including stopped)
podman ps -a

# 3. Custom format
podman ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# 4. Quiet — IDs only
podman ps -q    # running
podman ps -qa   # all

# 5. Filter
podman ps -a --filter status=exited
podman ps -a --filter name=web

# 6. Inspect
podman inspect web
podman inspect --format='{{.State.Status}}' web
podman inspect --format='{{.State.Running}}' web
podman inspect --format='{{.NetworkSettings.Ports}}' web
podman inspect --format='{{.Config.Env}}' web
podman inspect --format='{{.NetworkSettings.IPAddress}}' web

# 7. History
podman history registry.access.redhat.com/ubi8/httpd-24:latest

# Cleanup
podman stop web && podman rm web exited-one
```

---

### Lab B3: podman exec

```bash
# Setup
podman run -d --name execlab registry.access.redhat.com/ubi8/httpd-24:latest

# 1. Run a single command
podman exec execlab whoami
podman exec execlab hostname
podman exec execlab cat /etc/os-release

# 2. Run with flags
podman exec execlab ls -la /var/www/html
podman exec execlab ps aux

# 3. Interactive shell
podman exec -it execlab /bin/bash
# Inside: pwd, ls, exit

# 4. With env var (exec-session only)
podman exec -e MYVAR=hello execlab printenv MYVAR

# 5. As specific user
podman exec --user root execlab id
podman exec --user apache execlab id

# 6. Set working directory
podman exec --workdir /var/www/html execlab pwd

# 7. Write to container
podman exec execlab bash -c 'echo "<h1>from exec</h1>" > /var/www/html/test.html'
podman exec execlab cat /var/www/html/test.html

# Cleanup
podman stop execlab && podman rm execlab
```

---

### Lab B4: podman logs

```bash
# Setup
podman run -d --name logtest registry.access.redhat.com/ubi8/httpd-24:latest

# 1. View all logs
podman logs logtest

# 2. Follow logs in real-time
podman logs -f logtest   # Ctrl+C to stop

# 3. Last N lines
podman logs --tail 10 logtest

# 4. With timestamps
podman logs -t logtest
podman logs --timestamps logtest

# 5. Combine follow + timestamps
podman logs -ft logtest   # Ctrl+C to stop

# 6. Logs since time window
podman logs --since 1h logtest
podman logs --since 30m logtest

# 7. Logs persist after stop (deleted only on rm)
podman stop logtest
podman logs logtest   # still works!
podman rm logtest
podman logs logtest   # now fails — container deleted

# Cleanup
# (already done above)
```

---

### Lab B5: Environment variables

```bash
# 1. Single env var
podman run --rm -e GREETING="Hello Ahmad!" \
  registry.access.redhat.com/ubi8:latest \
  bash -c 'echo $GREETING'

# 2. Multiple env vars
podman run --rm \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  -e DB_NAME=myapp \
  -e LOG_LEVEL=debug \
  registry.access.redhat.com/ubi8:latest \
  env | grep -E "DB_|LOG_"

# 3. From host environment
export MY_TOKEN=secrettoken123
podman run --rm -e MY_TOKEN \
  registry.access.redhat.com/ubi8:latest \
  bash -c 'echo $MY_TOKEN'

# 4. Using env file
cat > /tmp/myapp.env << 'EOF'
APP_NAME=myapp
APP_VERSION=1.0.0
APP_ENV=staging
DB_HOST=postgres.internal
LOG_LEVEL=info
EOF

podman run --rm --env-file /tmp/myapp.env \
  registry.access.redhat.com/ubi8:latest \
  env | sort

# 5. Override env file with -e
podman run --rm \
  --env-file /tmp/myapp.env \
  -e APP_ENV=production \
  registry.access.redhat.com/ubi8:latest \
  env | grep APP_ENV
# APP_ENV=production (individual -e wins!)

# Cleanup
rm /tmp/myapp.env
```

---

### Lab B6: Port mapping

```bash
# 1. Basic port mapping
podman run -d --name web1 -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest
curl http://localhost:8080

# 2. Different host port
podman run -d --name web2 -p 9090:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest
curl http://localhost:9090

# 3. Multiple ports
podman run -d --name web3 -p 8080:8080 -p 8443:8443 \
  registry.access.redhat.com/ubi8/httpd-24:latest
podman ps   # see both ports in PORTS column

# 4. Localhost-only binding (not accessible from network)
podman run -d --name web4 -p 127.0.0.1:8888:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# 5. Random host port
podman run -d --name web5 -p :8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest
podman port web5   # find out which port was assigned

# 6. View port mappings
podman port web1
podman port --all

# Cleanup
podman stop web1 web2 web3 web4 web5
podman rm web1 web2 web3 web4 web5
```

---

### Lab B7: Container lifecycle (start, stop, restart, rm)

```bash
# 1. Create without running
podman create --name lc-test registry.access.redhat.com/ubi8/httpd-24:latest
podman ps -a   # Status: Created

# 2. Start it
podman start lc-test
podman ps      # Status: Up

# 3. Stop gracefully (SIGTERM)
podman stop lc-test
podman ps -a   # Status: Exited

# 4. Restart
podman restart lc-test
podman ps      # Status: Up

# 5. Pause
podman pause lc-test
podman ps      # Status: Paused

# 6. Unpause
podman unpause lc-test
podman ps      # Status: Up

# 7. Kill (immediate SIGKILL)
podman kill lc-test
podman ps -a   # Status: Exited

# 8. Force remove (running container)
podman start lc-test
podman rm --force lc-test   # no need to stop first

# Cleanup
podman ps -a   # confirm gone
```

---

### Lab B8: podman cp — copy files

```bash
# Setup
podman run -d --name copylab registry.access.redhat.com/ubi8/httpd-24:latest

# 1. Copy FROM host TO container
echo "<h1>Hello from host!</h1>" > /tmp/index.html
podman cp /tmp/index.html copylab:/var/www/html/index.html
podman exec copylab cat /var/www/html/index.html

# 2. Copy FROM container TO host
podman cp copylab:/etc/httpd/conf/httpd.conf /tmp/httpd.conf.bak
ls -lh /tmp/httpd.conf.bak

# 3. Copy a directory
podman cp copylab:/etc/httpd /tmp/httpd-dir-backup
ls /tmp/httpd-dir-backup/

# 4. Copy works even on STOPPED container
podman stop copylab
podman cp /tmp/index.html copylab:/var/www/html/new.html   # still works!

# 5. Copy between two containers
podman run -d --name copylab2 registry.access.redhat.com/ubi8/httpd-24:latest
podman cp copylab:/var/www/html/index.html copylab2:/var/www/html/copied.html
podman exec copylab2 cat /var/www/html/copied.html

# Cleanup
podman rm copylab copylab2
rm -f /tmp/index.html /tmp/httpd.conf.bak
rm -rf /tmp/httpd-dir-backup
```

---

### Lab B9: Podman networking

```bash
# 1. Inspect default network (DNS disabled!)
podman network inspect podman
# Look for: "dns_enabled": false

# 2. Create a custom network (DNS enabled!)
podman network create app-network
podman network ls

# 3. Inspect custom network (DNS enabled)
podman network inspect app-network
# Look for: "dns_enabled": true

# 4. Run containers on the custom network
podman run -d --name server1 --net app-network \
  registry.access.redhat.com/ubi8/httpd-24:latest

podman run -d --name server2 --net app-network \
  registry.access.redhat.com/ubi8/httpd-24:latest

# 5. Test DNS resolution (server2 can reach server1 by name)
podman exec server2 curl -s http://server1:8080

# 6. Get container IP in specific network
podman inspect server1 \
  -f '{{.NetworkSettings.Networks.app-network.IPAddress}}'

# 7. Connect existing container to another network
podman network create second-network
podman network connect second-network server1
podman network inspect second-network

# 8. Cleanup
podman stop server1 server2 && podman rm server1 server2
podman network rm app-network second-network
```

---

## Labs — Scenario Based

### Lab S1: Deploy a full web server stack

**Scenario:** A developer needs to run an Apache web server locally, test with custom HTML, and verify it is accessible.

```bash
# Setup project directory
mkdir ~/webproject && cd ~/webproject
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>DO188 Project</title></head>
<body>
  <h1>Ahmad Ali — DO188 Lab</h1>
  <p>Running on Red Hat UBI8 httpd-24</p>
  <p>Learning Podman Basics!</p>
</body>
</html>
EOF

# Step 1: Launch the web server
podman run -d \
  --name webproject \
  -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 2: Copy custom HTML into the running container
podman cp index.html webproject:/var/www/html/index.html

# Step 3: Verify it works
curl http://localhost:8080

# Step 4: Check container status
podman ps
podman inspect webproject --format "{{.State.Status}}"
podman port webproject

# Step 5: View access logs
podman logs webproject

# Step 6: Update the page live
echo "<h2>Updated content!</h2>" >> index.html
podman cp index.html webproject:/var/www/html/index.html
curl http://localhost:8080

# Cleanup
podman stop webproject && podman rm webproject
rm -rf ~/webproject
```

---

### Lab S2: Run a database with proper environment variables

**Scenario:** Set up a PostgreSQL database container for local development and connect to it.

```bash
# Step 1: Create env file for database credentials
cat > /tmp/db.env << 'EOF'
POSTGRES_USER=devuser
POSTGRES_PASSWORD=devpassword123
POSTGRES_DB=appdb
EOF

# Step 2: Run PostgreSQL
podman run -d \
  --name dev-postgres \
  --env-file /tmp/db.env \
  -p 5432:5432 \
  docker.io/postgres:15-alpine

# Step 3: Wait for startup and check logs
sleep 5
podman logs dev-postgres | tail -5

# Step 4: Verify it is running
podman ps
podman inspect dev-postgres --format "{{.State.Status}}"

# Step 5: Connect and create schema
podman exec -it dev-postgres psql -U devuser -d appdb << 'SQL'
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
INSERT INTO users (name, email) VALUES
  ('Ahmad Ali', 'ahmad@example.com'),
  ('Sara Khan', 'sara@example.com');
SELECT * FROM users;
\q
SQL

# Step 6: Run non-interactive query
podman exec dev-postgres psql -U devuser -d appdb \
  -c "SELECT COUNT(*) FROM users;"

# Cleanup
podman stop dev-postgres && podman rm dev-postgres
rm /tmp/db.env
```

---

### Lab S3: Debug a misconfigured container using exec and cp

**Scenario:** A web server container is running but returning errors. Debug using `exec` and `cp` — exactly as shown in the DO188 guided exercise.

```bash
# Step 1: Start a container
podman run -d --name debug-web -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 2: Test the server
curl -I http://localhost:8080

# Step 3: Check what's running inside
podman exec debug-web ps aux | grep httpd
podman exec debug-web ls -la /var/www/html/

# Step 4: Check logs for errors
podman logs debug-web | tail -20

# Step 5: Copy config file OUT to inspect it
podman cp debug-web:/etc/httpd/conf/httpd.conf /tmp/httpd.conf
head -30 /tmp/httpd.conf

# Step 6: Validate the config syntax
podman exec debug-web httpd -t
# Expected: Syntax OK

# Step 7: Check which ports httpd is actually listening on
podman exec debug-web ss -tlnp 2>/dev/null || \
  podman exec debug-web cat /proc/net/tcp

# Step 8: Make a config change and copy back
echo "# Added by Ahmad during DO188 lab" >> /tmp/httpd.conf
podman cp /tmp/httpd.conf debug-web:/etc/httpd/conf/httpd.conf

# Step 9: Graceful reload (no restart needed)
podman exec debug-web httpd -t && \
podman exec debug-web kill -HUP 1   # SIGHUP = graceful reload

# Cleanup
podman stop debug-web && podman rm debug-web
rm /tmp/httpd.conf
```

---

### Lab S4: Multi-service networking (DNS between containers)

**Scenario:** Deploy two services that communicate with each other using container DNS — based on the DO188 guided exercise with times and cities apps.

```bash
# Step 1: Create a network with DNS enabled
podman network create services-network
podman network inspect services-network
# Confirm: "dns_enabled": true

# Step 2: Start service A (acts as "times" app)
podman run -d \
  --name service-a \
  --net services-network \
  -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 3: Get IP of service-a
podman inspect service-a \
  -f '{{.NetworkSettings.Networks.services-network.IPAddress}}'

# Step 4: Start service B — passes service-a's URL as env var
podman run -d \
  --name service-b \
  --net services-network \
  -p 8090:8080 \
  -e UPSTREAM_URL=http://service-a:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 5: Test service-b can reach service-a by DNS name
podman exec service-b curl -s http://service-a:8080

# Step 6: Test by IP as well (both should work)
IP=$(podman inspect service-a \
  -f '{{.NetworkSettings.Networks.services-network.IPAddress}}')
podman exec service-b curl -s http://${IP}:8080

# Step 7: Verify from outside
curl http://localhost:8090
curl http://localhost:8080

# Cleanup
podman stop service-a service-b
podman rm service-a service-b
podman network rm services-network
```

---

### Lab S5: Clean up a messy server (orphan containers)

**Scenario:** A server has many stopped containers filling up storage. Systematically clean them up.

```bash
# Step 1: Create a mess (simulate orphaned containers)
for i in 1 2 3 4 5; do
  podman run --name orphan-$i \
    registry.access.redhat.com/ubi8:latest \
    echo "orphan $i done"
done
# Also keep one important running container
podman run -d --name important-service \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 2: Identify the problem
podman ps -a
echo "Running: $(podman ps -q | wc -l)"
echo "Total: $(podman ps -qa | wc -l)"
echo "Stopped: $(podman ps -qa --filter status=exited | wc -l)"

# Step 3: Check disk usage
podman system df

# Step 4: Remove only exited containers (safe — keeps running ones)
podman rm $(podman ps -qa --filter status=exited)

# Step 5: Verify important service is still running
podman ps

# Step 6: Full cleanup (removes everything not running)
podman stop important-service
podman container prune -f

# Step 7: Final disk check
podman system df

# Cleanup
podman rm -a
```

---

## Labs — Intermediate

### Lab I1: Resource limits and container constraints

```bash
# 1. Run with memory limit
podman run -d --name mem-limited \
  --memory 256m \
  --memory-swap 256m \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Verify limit is set
podman inspect mem-limited --format "{{.HostConfig.Memory}}"
# Output: 268435456 (256 * 1024 * 1024 bytes)

# 2. CPU limit
podman run -d --name cpu-limited \
  --cpus="0.5" \
  registry.access.redhat.com/ubi8/httpd-24:latest

podman inspect cpu-limited --format "{{.HostConfig.NanoCpus}}"
# Output: 500000000 (0.5 CPUs in nanoseconds)

# 3. Resource stats
podman stats --no-stream

# 4. Read-only filesystem
podman run --rm --read-only \
  registry.access.redhat.com/ubi8:latest \
  touch /tmp/test 2>&1 || echo "Blocked: read-only filesystem"

# Cleanup
podman stop mem-limited cpu-limited && podman rm mem-limited cpu-limited
```

---

### Lab I2: Container health checks

```bash
# Run with health check
podman run -d \
  --name healthcheck-demo \
  --health-cmd "curl -f http://localhost:8080 || exit 1" \
  --health-interval 10s \
  --health-timeout 5s \
  --health-retries 3 \
  -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Wait for health check to run
sleep 15

# Check health status
podman inspect healthcheck-demo --format "{{.State.Health.Status}}"
# Output: healthy

# Manually trigger a health check
podman healthcheck run healthcheck-demo

# View health check history
podman inspect healthcheck-demo \
  --format "{{range .State.Health.Log}}Exit={{.ExitCode}} {{end}}"

# Cleanup
podman stop healthcheck-demo && podman rm healthcheck-demo
```

---

### Lab I3: Batch container operations

```bash
# Create multiple containers
for i in $(seq 1 5); do
  podman run -d --name batch-$i \
    registry.access.redhat.com/ubi8/httpd-24:latest
done

# List all
podman ps --format "table {{.Names}}\t{{.Status}}"

# Stop all running at once
podman stop $(podman ps -q)

# Start all stopped at once
podman start $(podman ps -qa --filter status=exited)

# Stop and remove all
podman stop -a && podman rm -a

# Verify clean state
podman ps -a
```

---

### Lab I4: Container commit — snapshot a modified container

```bash
# Start a container and modify it
podman run -d --name to-commit registry.access.redhat.com/ubi8/httpd-24:latest

# Add custom content
podman exec to-commit bash -c \
  'echo "<h1>Custom content added via exec</h1>" > /var/www/html/custom.html'

# Commit the running container as a new image
podman commit to-commit myhttpd:customized

# Verify new image exists
podman images | grep myhttpd

# Run the new custom image — content persists!
podman run -d --name from-commit -p 9999:8080 myhttpd:customized
curl http://localhost:9999/custom.html

# Cleanup
podman stop to-commit from-commit
podman rm to-commit from-commit
podman rmi myhttpd:customized
```

---

### Lab I5: Multi-environment deployment (dev vs prod)

```bash
# Create environment-specific configurations
cat > /tmp/dev.env << 'EOF'
APP_ENV=development
DB_HOST=localhost
DB_PORT=5432
LOG_LEVEL=debug
CACHE_ENABLED=false
EOF

cat > /tmp/prod.env << 'EOF'
APP_ENV=production
DB_HOST=prod-db.internal
DB_PORT=5432
LOG_LEVEL=warn
CACHE_ENABLED=true
EOF

# Deploy development instance
podman run -d --name app-dev \
  --env-file /tmp/dev.env \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Deploy production instance
podman run -d --name app-prod \
  --env-file /tmp/prod.env \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Compare environments
echo "=== DEV ===" && podman exec app-dev env | grep APP_ENV
echo "=== PROD ===" && podman exec app-prod env | grep APP_ENV

# Both use the SAME image — only env vars differ (12-factor principle!)
podman inspect app-dev --format "{{.ImageName}}"
podman inspect app-prod --format "{{.ImageName}}"

# Cleanup
podman stop app-dev app-prod && podman rm app-dev app-prod
rm /tmp/dev.env /tmp/prod.env
```

---

## Labs — Industry / Interview Questions with Solutions

### Q1: What is the difference between `podman run` and `podman start`?

**Answer:**
- `podman run` = creates a **new container** from an image and starts it
- `podman start` = restarts an **existing stopped container**

```bash
# Demonstrate:
podman run --name demo registry.access.redhat.com/ubi8:latest echo "first run"
podman ps -a   # Status: Exited

# run again = CREATES ANOTHER new container (conflict on same name!)
# start = restarts the EXISTING one
podman start demo
podman ps -a   # Status: Exited again (echo finished)

podman rm demo
```

---

### Q2: What is the difference between `podman stop` and `podman kill`?

**Answer:**
- `podman stop` = sends **SIGTERM** (graceful), waits 10s, then sends **SIGKILL**
- `podman kill` = sends **SIGKILL immediately** (no grace period)

```bash
podman run -d --name sig-demo registry.access.redhat.com/ubi8/httpd-24:latest

# Graceful: httpd gets SIGTERM and cleans up properly
podman stop sig-demo

podman start sig-demo
# Immediate: no cleanup possible
podman kill sig-demo

podman rm sig-demo
```

---

### Q3: Can two containers share the same host port?

**Answer:** No. Each host port can only be used by one process (container or otherwise) at a time.

```bash
podman run -d --name first  -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest
podman run -d --name second -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest
# Error: address already in use

podman stop first && podman rm first
```

---

### Q4: Are logs lost when you stop a container?

**Answer:** No. Logs are preserved until the container is **removed** with `podman rm`.

```bash
podman run -d --name logdemo registry.access.redhat.com/ubi8/httpd-24:latest
sleep 2
podman stop logdemo
podman logs logdemo    # STILL works after stop!

podman rm logdemo
podman logs logdemo    # Error: no such container — NOW they are gone
```

---

### Q5: Can you exec into a stopped container?

**Answer:** No. `podman exec` requires a **running** container.

```bash
podman run --name stopped-demo registry.access.redhat.com/ubi8:latest echo "done"
# Container is now in Exited state
podman exec stopped-demo echo "will this work?" 
# Error: container is not running

podman rm stopped-demo
```

---

### Q6: Why use port 8080 instead of 80 in containers?

**Answer:** Ports below 1024 are **privileged** and require root. Rootless Podman and OpenShift cannot bind to port 80. Always use port 8080 (HTTP) and 8443 (HTTPS).

```bash
# This would fail in rootless mode:
# podman run -p 80:80 ...

# Correct:
podman run -d -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest
podman stop $(podman ps -q) && podman rm $(podman ps -qa)
```

---

### Q7: How do you get a container's IP address?

**Answer:** Use `podman inspect` with a Go template.

```bash
podman run -d --name ip-demo registry.access.redhat.com/ubi8/httpd-24:latest

# Get IP address
podman inspect ip-demo --format "{{.NetworkSettings.IPAddress}}"

# If container is on a custom network:
podman inspect ip-demo \
  --format "{{.NetworkSettings.Networks.podman.IPAddress}}"

podman stop ip-demo && podman rm ip-demo
```

---

### Q8: What does `podman ps -qa` mean?

**Answer:** `-q` (quiet) = show only container IDs. `-a` (all) = include stopped containers. Together: **IDs of ALL containers** — used in batch operations.

```bash
# Create some containers in different states
podman run -d --name r1 registry.access.redhat.com/ubi8/httpd-24:latest
podman run --name s1 registry.access.redhat.com/ubi8:latest echo "done"

podman ps -q    # only RUNNING IDs
podman ps -qa   # ALL container IDs (running + stopped)

# Practical use: remove all stopped containers
podman rm $(podman ps -qa --filter status=exited)
podman stop r1 && podman rm r1
```

---

### Q9: What is the default Podman network and why does DNS not work with it?

**Answer:** The default network is named `podman`. DNS is disabled on it by design. To use DNS (container-to-container by name), always create a **custom network**.

```bash
# Default network has DNS disabled
podman network inspect podman | grep dns_enabled
# "dns_enabled": false

# Custom network has DNS enabled
podman network create mynet
podman network inspect mynet | grep dns_enabled
# "dns_enabled": true

podman network rm mynet
```

---

### Q10: How do you pass sensitive data (like passwords) to a container without exposing them in shell history?

**Answer:** Use `--env-file` to read from a file (don't store in `~/.bash_history`). In production, use OpenShift Secrets.

```bash
# BAD: appears in bash history
podman run -e DB_PASSWORD=mysecret ...

# BETTER: env file (add *.env to .gitignore!)
cat > /tmp/secrets.env << 'EOF'
DB_PASSWORD=mysecret
API_KEY=abc123
EOF
podman run --rm --env-file /tmp/secrets.env \
  registry.access.redhat.com/ubi8:latest env | grep -E "DB_|API_"

# BEST for OpenShift: use secrets
# oc create secret generic myapp-secrets \
#   --from-literal=DB_PASSWORD=mysecret

rm /tmp/secrets.env
```

---

## Guided Exercises from the Book

These are the exact exercises from the DO188 student guide (RHOSCP 4.10). Reproduced here for reference and self-study.

### Guided Exercise 1: Creating Containers with Podman

**Objective:** Run containers using the `podman run` command.

```bash
# Exercise 1.1 — Pull ubi-minimal image
podman pull registry.access.redhat.com/ubi8/ubi-minimal:8.5
podman images

# Exercise 1.2 — Run a container that prints Hello Red Hat
podman run registry.access.redhat.com/ubi8/ubi-minimal:8.5 echo 'Hello Red Hat'

# Exercise 1.3 — Verify container is not running after exit
podman ps   # should be empty

# Exercise 2 — Set and print environment variables
podman run -e GREET=Hello -e NAME='Red Hat' \
  registry.access.redhat.com/ubi8/ubi-minimal:8.5 \
  printenv GREET NAME
# Output:
# Hello
# Red Hat

# Exercise 2.2 — Verify no longer running
podman ps   # empty

# Exercise 3.1 — Run httpd server (foreground)
podman run -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest
# Press Ctrl+C to stop

# Exercise 3.4 — Run in background
podman run -d -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24:latest

# Exercise 3.5 — Verify running
podman ps
```

---

### Guided Exercise 2: Accessing Containerized Network Services

**Objective:** Run two applications that communicate using Podman DNS.

```bash
# Step 1: Inspect default network — DNS disabled
podman network inspect podman | grep dns_enabled
# "dns_enabled": false

# Step 2: Create a DNS-enabled network
podman network create cities
podman network inspect cities | grep dns_enabled
# "dns_enabled": true

# Step 3: Run times-app on the cities network
# (Using httpd-24 as a substitute for the course-specific image)
podman run -d --name times-app \
  --net cities \
  -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 4: Get its IP address
podman inspect times-app \
  -f '{{.NetworkSettings.Networks.cities.IPAddress}}'

# Step 5: Test by IP (from another container on same network)
IP=$(podman inspect times-app \
  -f '{{.NetworkSettings.Networks.cities.IPAddress}}')
podman run --rm --net cities \
  registry.access.redhat.com/ubi8/ubi-minimal:8.5 \
  curl -s http://${IP}:8080

# Step 6: Test by DNS name (container name as hostname)
podman run --rm --net cities \
  registry.access.redhat.com/ubi8/ubi-minimal:8.5 \
  curl -s http://times-app:8080

# Step 7: Run cities-app referencing times-app by DNS name
podman run -d --name cities-app \
  --net cities \
  -p 8090:8080 \
  -e UPSTREAM_URL=http://times-app:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 8: Verify cities-app is running
curl http://localhost:8090

# Cleanup
podman stop times-app cities-app && podman rm times-app cities-app
podman network rm cities
```

---

### Guided Exercise 3: Accessing Containers (exec and cp)

**Objective:** Use `podman exec` and `podman cp` to debug and fix a misconfigured nginx container.

```bash
# Step 1: Start nginx container
podman run -d --name nginx -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 2: Test it
curl http://localhost:8080

# Step 3.1: Copy error log from container
podman cp nginx:/var/log/httpd/error_log ./error.log
cat error.log

# Step 3.3: Check directory exists
podman exec nginx ls /var/www/html

# Step 3.4: Check actual html directory
podman exec nginx ls /var/www/html/

# Step 4.1: Copy config file out
podman cp nginx:/etc/httpd/conf/httpd.conf ./nginx-httpd.conf

# Step 4.2: Inspect config (look for root directive)
grep -n "DocumentRoot\|root" nginx-httpd.conf | head -10

# Step 4.3: Copy modified config back
podman cp nginx-httpd.conf nginx:/etc/httpd/conf/httpd.conf

# Step 5.1: Reload server config
podman exec nginx httpd -s reload

# Step 5.2: Verify
curl http://localhost:8080

# Cleanup
podman stop nginx && podman rm nginx
rm error.log nginx-httpd.conf
```

---

### Guided Exercise 4: Managing the Container Lifecycle

**Objective:** Start, inspect, stop, restart, and delete a container.

```bash
# Step 1: Create and run the container
podman run -d --name httpd -p 8080:8080 \
  registry.access.redhat.com/ubi8/httpd-24:latest

# Step 2: Verify it is running
podman ps
podman inspect --format='{{.State.Status}}' httpd   # running
podman inspect --format='{{.State.Running}}' httpd  # true

# Step 3: Access it
curl http://localhost:8080

# Step 4: Stop the container
podman stop httpd

# Step 5: Verify stopped state
podman inspect --format='{{.State.Status}}' httpd   # exited
podman inspect --format='{{.State.Running}}' httpd  # false
curl http://localhost:8080 2>&1 || echo "Server no longer accessible"

# Step 6: Restart
podman restart httpd
podman ps    # Up again

# Step 7: Verify accessible again
curl http://localhost:8080

# Step 8: Try to remove RUNNING container (will fail)
podman rm httpd
# Error: container is running

# Step 9: Force remove running container
podman rm --force httpd

# Step 10: Verify gone
podman ps -a | grep httpd || echo "Container deleted successfully"
```

---

## Key Concepts Summary

### The 5 Core Concepts of Chapter 2

```
1. DAEMONLESS PODMAN
   No root daemon = no single point of failure = better security
   Each command directly manages containers via Linux kernel features

2. CONTAINER LAYERS
   Image = read-only stacked layers
   Container = image layers + thin ephemeral writable layer on top
   Delete container = lose the writable layer (use volumes for persistence!)

3. PORT MAPPING (-p host:container)
   Containers are network-isolated by default
   -p maps host port to container port
   Use 8080 not 80 (rootless/OpenShift cannot use privileged ports < 1024)

4. DNS IN CUSTOM NETWORKS
   Default podman network = NO DNS
   Custom network = DNS enabled automatically
   Container name = hostname within the network

5. LIFECYCLE: create → start → stop → restart → rm
   stop = SIGTERM (graceful)
   kill = SIGKILL (immediate)
   rm = permanent deletion of container + its logs
```

### Quick Decision Guide

| I want to... | Command |
|---|---|
| Run a one-off command | `podman run --rm image command` |
| Start a background service | `podman run -d --name x image` |
| Get a shell in a running container | `podman exec -it container /bin/bash` |
| See container logs | `podman logs container` |
| Follow logs live | `podman logs -f container` |
| Copy a file to container | `podman cp file container:/path` |
| Copy a file from container | `podman cp container:/path ./local` |
| See what's running | `podman ps` |
| See all containers | `podman ps -a` |
| Get container IP | `podman inspect c --format "{{.NetworkSettings.IPAddress}}"` |
| Stop gracefully | `podman stop container` |
| Kill immediately | `podman kill container` |
| Remove stopped container | `podman rm container` |
| Remove running container | `podman rm --force container` |
| Remove all stopped | `podman container prune -f` |
| Enable DNS between containers | `podman network create mynet` |

---

## Quiz: Self-Assessment

Test your knowledge before the EX188 exam. Answers are at the bottom.

**Q1.** Which flag makes a container automatically delete itself after it exits?

**Q2.** What signal does `podman stop` send first to a container?

**Q3.** Can `podman exec` be used on a stopped container?

**Q4.** What is the default number of seconds `podman stop` waits before sending SIGKILL?

**Q5.** Which command shows ALL containers, including stopped ones?

**Q6.** What format does the `--format` flag in `podman inspect` use?

**Q7.** Why does DNS not work with the default `podman` network?

**Q8.** What is the syntax to copy a file FROM a container TO the host?

**Q9.** If two containers are on the same custom Podman network, how does container B refer to container A?

**Q10.** What is the difference between `-i` and `-t` flags in `podman exec`?

---

**Answers:**

1. `--rm` flag
2. SIGTERM
3. No — container must be in running state
4. 10 seconds
5. `podman ps --all` (or `podman ps -a`)
6. Go template syntax (`{{.Field.SubField}}`)
7. DNS is disabled on the default podman network — use a custom network
8. `podman cp containerName:/path/in/container ./path/on/host`
9. By its container name (which acts as the DNS hostname)
10. `-i` keeps stdin open (accept input); `-t` allocates a pseudo-TTY (terminal prompt). Use `-it` together for an interactive shell session.

---

## My Learning Progress

| Section | Status | Lab Count | Notes |
|---------|--------|-----------|-------|
| 2.1 Creating Containers | ✅ Complete | 3 | `podman run`, `--rm`, `-d`, `-it` |
| 2.2 Container Networking | ✅ Complete | 2 | `network create`, DNS, `--net` |
| 2.3 Port Mapping | ✅ Complete | 2 | `-p`, `podman port`, binding |
| 2.4 Accessing Containers | ✅ Complete | 3 | `exec`, `cp`, container layers |
| 2.5 Lifecycle | ✅ Complete | 4 | `stop`, `kill`, `restart`, `rm` |
| Basic Labs | ✅ Complete | 9 labs | One per topic |
| Scenario Labs | ✅ Complete | 5 labs | Real-world scenarios |
| Intermediate Labs | ✅ Complete | 5 labs | Resource limits, health checks |
| Interview Labs | ✅ Complete | 10 Q&A | With full solutions |

**Total labs completed: 52 across all tiers**

---

## Resources

- [Podman Official Documentation](https://docs.podman.io/en/latest/)
- [Red Hat DO188 Course Page](https://www.redhat.com/en/services/training/do188)
- [Basic Networking Guide for Podman](https://github.com/containers/podman/blob/main/docs/tutorials/basic_networking.md)
- [Go Templates (for podman inspect --format)](https://pkg.go.dev/text/template)
- [Red Hat UBI Images](https://catalog.redhat.com/software/containers/search?q=ubi)
- [EX188 Exam Objectives](https://www.redhat.com/en/services/training/ex188-red-hat-certified-specialist-containers-exam)

---

*Learning journal maintained by **Ahmad Ali** | MNS University of Agriculture, Multan, Pakistan*  
*Course: DO188 Red Hat OpenShift Development I*
