# DO188 — Chapter 3: Container Images
### Red Hat OpenShift Development I: Introduction to Containers with Podman

> **Study Guide & Lab Journal** | Ahmad Ali | MNS University of Agriculture, Multan  
> **Course:** DO188 (RHOSCP 4.10) | **Target Exam:** EX188 — Red Hat Certified Specialist in Containers  
> **Mentor:** [Ali Ijaz](https://www.linkedin.com/in/aliijaz/) | **LinkedIn:** [Ahmad Ali](https://linkedin.com/in/ahmad-ali)

---

## Table of Contents

1. [Chapter Overview & Goal](#chapter-overview--goal)
2. [Chapter Roadmap](#chapter-roadmap)
3. [Section 1 — Container Image Registries](#section-1--container-image-registries)
   - [What is a Container Registry?](#what-is-a-container-registry)
   - [Red Hat Registries](#red-hat-registries)
   - [Universal Base Images (UBI)](#universal-base-images-ubi)
   - [Quay.io](#quayio)
   - [Managing Registries with Podman](#managing-registries-with-podman)
   - [Registry Authentication](#registry-authentication)
4. [Section 2 — Managing Images](#section-2--managing-images)
   - [Image Versioning and Tags](#image-versioning-and-tags)
   - [Searching for Images](#searching-for-images)
   - [Pulling Images](#pulling-images)
   - [Building Images](#building-images)
   - [Pushing Images](#pushing-images)
   - [Inspecting Images](#inspecting-images)
   - [Tagging Images](#tagging-images)
   - [Removing Images](#removing-images)
5. [Complete Command Reference](#complete-command-reference)
6. [Labs — Basic (per topic)](#labs--basic-per-topic)
7. [Labs — Scenario Based](#labs--scenario-based)
8. [Labs — Intermediate](#labs--intermediate)
9. [Labs — Industry / Interview Questions with Solutions](#labs--industry--interview-questions-with-solutions)
10. [Security Perspective](#security-perspective)
11. [Enterprise Best Practices](#enterprise-best-practices)
12. [Troubleshooting Guide](#troubleshooting-guide)
13. [Interview Preparation](#interview-preparation)
14. [Certification Exam Tips](#certification-exam-tips)
15. [Key Concepts Summary](#key-concepts-summary)
16. [Chapter Summary & Cheat Sheet](#chapter-summary--cheat-sheet)

---

## Chapter Overview & Goal

**Chapter Goal:** Navigate container registries to find and manage container images.

**Objectives:**
- Navigate container registries (Red Hat Registry, Quay.io, Docker Hub)
- Pull and manage container images using Podman
- Authenticate with private registries
- Configure Podman registry settings
- Tag, inspect, push, and remove images

**Sections in the book:**
| Section | Topic | Exercise |
|---------|-------|----------|
| 3.1 | Container Image Registries | Guided Exercise: images-basics |
| 3.2 | Managing Images | Guided Exercise: images-managing |
| — | Lab | Container Images (images-lab) |

---

## Chapter Roadmap

```
Chapter 3: Container Images
│
├── Section 3.1 — Container Image Registries
│   ├── What is a container registry?
│   ├── The Containerfile (brief intro)
│   ├── Red Hat Registry (registry.access.redhat.com + registry.redhat.io)
│   ├── Red Hat Catalog (catalog.redhat.com)
│   ├── Useful Red Hat images (UBI8, Python-39, Node.js-16, Go Toolset)
│   ├── Universal Base Images (UBI) — what they are and why
│   ├── Quay.io — community registry
│   ├── Managing registries with Podman (registries.conf)
│   │   ├── Qualified vs unqualified image names
│   │   ├── unqualified-search-registries configuration
│   │   └── Blocking a registry
│   └── Managing registry credentials
│       ├── podman login
│       ├── auth.json credential storage
│       └── base64 encoding of credentials
│
├── Section 3.2 — Managing Images
│   ├── Image versioning and semantic versioning (MAJOR.MINOR.PATCH)
│   ├── Image tags — format and naming
│   ├── The :latest tag — why it is a bad practice
│   ├── podman search — find images
│   ├── podman pull / podman image pull — download images
│   │   └── Image storage: ~/.local/share/containers (user) vs /var/lib/containers (root)
│   ├── podman build — build from Containerfile
│   ├── podman push — upload to registry
│   ├── podman image inspect / podman inspect — image metadata
│   │   └── --format with Go templates
│   ├── podman image tag — create additional tags
│   ├── podman image rm / podman rmi — remove images
│   │   ├── -f (force) flag
│   │   └── --all flag
│   └── podman image prune — remove dangling/unused images
│       ├── Dangling images (untagged, unreferenced)
│       ├── -a / --all flag
│       └── -f / --force flag
│
└── Lab: Container Images
    ├── Build image on Quay using Containerfile
    ├── Pull from Quay
    ├── Tag the image
    └── Run the tagged container
```

---

## Section 1 — Container Image Registries

### What is a Container Registry?

> **From the DO188 book:** "A container image is a packaged version of your application, with all the dependencies necessary for the application to run. You can use image registries to store container images to later share them in a controlled manner."

**Definition:** A container registry is a server-side application that stores, manages, and distributes container images. Think of it like GitHub — but instead of storing code, it stores built application images.

**Real analogy:**  
- Registry = App Store (stores the packages)  
- Image = App (the installable package)  
- Container = Running app on your phone  

**Examples of registries:**
| Registry | URL | Auth Required | Use Case |
|----------|-----|---------------|----------|
| Red Hat Registry | registry.access.redhat.com | ❌ Free | UBI images, certified content |
| Red Hat Registry (Pro) | registry.redhat.io | ✅ RH account | Full RHEL product images |
| Quay.io | quay.io | Optional (for private) | Store your own images |
| Docker Hub | docker.io | Optional | Community images |
| Amazon ECR | *.amazonaws.com | ✅ AWS credentials | AWS deployments |
| GitHub Packages | ghcr.io | ✅ GitHub token | GitHub CI/CD pipelines |

**How a pull works — step by step:**

```bash
podman pull registry.redhat.io/ubi8/ubi:8.6
# Step 1: Podman resolves registry.redhat.io hostname via DNS
# Step 2: Checks auth.json for saved credentials
# Step 3: Downloads the image manifest (index.json)
# Step 4: Downloads each image layer (blobs)
# Step 5: Assembles layers into a local image
# Step 6: Stores in ~/.local/share/containers/
```

---

### The Containerfile (brief intro from the book)

Chapter 3 introduces the Containerfile concept because it explains how images are BUILT. Detailed coverage is in Chapter 4.

**Key points from the book:**

```dockerfile
FROM registry.redhat.io/ubi9
CMD echo "Hello world"
```

- `FROM` — specifies the **parent/base image** to inherit from
- `CMD` — the command that runs when the container starts
- Images are built in **layers** — each instruction = one layer
- Child images **inherit** all layers from the parent

> **Book quote:** "Usually, container images are created based on other container images. For example, an image that outputs Hello world might be created by using a Linux parent image."

---

### Red Hat Registries

Red Hat provides **two separate registries** with different access models:

```
registry.access.redhat.com   ← FREE, no login required
├── ubi8/ubi:latest           (Universal Base Image 8)
├── ubi8/python-39:latest     (Python 3.9 on UBI8)
├── ubi8/nodejs-16:1          (Node.js 16 on UBI8)
└── ubi8/go-toolset           (Go on UBI8)

registry.redhat.io            ← REQUIRES Red Hat account
├── rhel8/httpd-24            (Full RHEL8 httpd)
├── rhel8/mariadb-103:1       (MariaDB on RHEL8)
└── rhel9/python-39           (Python on RHEL9)
```

**Red Hat Catalog:** https://catalog.redhat.com/software/containers/explore

The catalog provides for each image:
- **Overview** — general info, description
- **Security** — Health Index grade (A to F), CVE scan results
- **Technical Information** — architecture, exposed ports
- **Packages** — all installed RPM packages
- **Dockerfile** — the exact Containerfile used to build the image
- **Get this image** — pull commands for Podman, Docker, and RHOCP

**What senior engineers know:**
- Always check the **Security tab** before using an image in production
- The Health Index grade directly impacts your org's security posture
- Images graded C or below should trigger a security review
- Use the **Packages tab** to audit what software your base image includes

---

### Universal Base Images (UBI)

> **From the book:** "Red Hat UBIs are Open Container Initiative (OCI) compliant images that contain portions of Red Hat Enterprise Linux (RHEL). UBIs are enterprise grade container images that are engineered to be the base operating system layer for your containerized applications."

**Key UBI facts you must know for the exam:**

| Fact | Detail |
|------|--------|
| License | Freely distributable — no Red Hat subscription needed |
| Content | Subset of RHEL packages |
| Support | Full Red Hat support ONLY when deployed on RHOCP/RHEL |
| Compliance | OCI compliant |
| Usability | Can be pushed to ANY registry, deployed on ANY platform |
| Package manager | Uses DNF (full UBI) or microdnf (minimal UBI) |

**UBI variants:**

```bash
# Full UBI8 — has dnf, bash, all standard tools (~200MB)
registry.access.redhat.com/ubi8/ubi:8.6
registry.access.redhat.com/ubi8:latest

# Minimal UBI8 — smaller, has microdnf, no bash (~90MB)
registry.access.redhat.com/ubi8/ubi-minimal:8.5

# Micro UBI — smallest, no package manager at all (~13MB)
# For base image of compiled languages (Go, Rust)

# Init UBI — for systemd-based containers
registry.access.redhat.com/ubi8/ubi-init

# Language runtime images based on UBI8:
registry.access.redhat.com/ubi8/python-39:latest
registry.access.redhat.com/ubi8/nodejs-16:1
registry.access.redhat.com/ubi8/go-toolset
```

---

### Quay.io

> **From the book:** "Although the Red Hat Registry only stores images from Red Hat and certified providers, you can use Quay.io, another image registry, to store your own images. Storing public images in Quay is free, but there are some options only available for paying customers."

**Key Quay.io facts:**
- Login using your **Red Hat developer account**
- Public image storage is **free**
- Private repositories require a paid plan
- Red Hat offers **Quay Enterprise** (self-hosted, on-premise)
- Quay has built-in **image security scanning**
- Images can be built directly in Quay using a **Containerfile** (no local build needed)

**Creating an image on Quay:**
1. Login to quay.io
2. Click "Create New Repository"
3. Fill in: Name, Visibility (Public), Initialize from Dockerfile
4. Upload your Containerfile → Quay builds the image automatically

---

### Managing Registries with Podman

#### Qualified vs Unqualified Image Names

```bash
# QUALIFIED name — includes full registry URL (recommended!)
registry.redhat.io/rhel7/rhel:7.9
registry.access.redhat.com/ubi8/python-39:latest
quay.io/argoproj/argocd:latest

# UNQUALIFIED name — no registry URL (Podman searches registries.conf)
ubi8/python-39
rhel7/rhel:7.9
nginx
```

> **Book quote:** "If you do not provide the registry URL, Podman does not know which registry to use, so it relies on the /etc/containers/registries.conf file."

#### The registries.conf File

```bash
# View current registry configuration
cat /etc/containers/registries.conf

# Key configuration: search order for unqualified names
unqualified-search-registries = ['registry.redhat.io', 'docker.io']
# Podman tries registry.redhat.io first, then docker.io

# Block a specific registry
[[registry]]
location="docker.io"
blocked=true
# Now: podman pull nginx  →  Error: docker.io is blocked
```

**User-level override** (no root required):
```bash
# Create user-level registry config
mkdir -p ~/.config/containers/
cat > ~/.config/containers/registries.conf << 'EOF'
unqualified-search-registries = ['registry.access.redhat.com', 'quay.io']
EOF
```

**On Windows/Mac (Podman Machine):**
```bash
# Access the Linux VM that runs Podman
podman machine ssh
# Then find registries.conf inside the VM
ls /etc/containers/containers.conf
```

---

### Registry Authentication

```bash
# Login to registry.redhat.io (requires Red Hat account)
podman login registry.redhat.io
# Username: YOUR_USER
# Password: YOUR_PASSWORD   ← hidden as you type (no feedback shown)
# Login Succeeded!

# Login to Quay.io
podman login quay.io
# Username: YOUR_QUAY_USER
# Password:
# Login Succeeded!

# Check current login status
podman login --get-login registry.redhat.io
# Output: YOUR_USER

# Logout
podman logout registry.redhat.io
```

**Where credentials are stored:**

```bash
# Credentials file location
cat ${XDG_RUNTIME_DIR}/containers/auth.json
# Output:
# {
#   "auths": {
#     "registry.redhat.io": {
#       "auth": "dXNlcjpodW50ZXIy"
#     }
#   }
# }

# Decode the base64 credentials to see what is stored
echo -n "dXNlcjpodW50ZXIy" | base64 -d
# Output: user:hunter2   (username:password)
```

> **Security warning:** The auth.json file stores credentials in BASE64 — this is ENCODING not ENCRYPTION. Anyone with access to this file can decode your credentials. Protect this file!

**Common authentication errors and solutions:**

```
Error: initializing source docker://registry.redhat.io/...: 
       unable to retrieve auth token: invalid username/password

Solution: podman login registry.redhat.io
```

```
Error: unauthorized: access to the requested resource is not authorized

Solution: Check if your account has entitlement for this specific image
          UBI images (registry.access.redhat.com) are free, no login needed
```

---

## Section 2 — Managing Images

### Image Versioning and Tags

> **From the book:** "Because container images package software, they become deployment artifacts in their own right. To help keep images up to date, you should version them with the intention of mapping the image versions to the product versions."

**Image name full format:**
```
[<image repository>/<namespace>/]<image name>[:<tag>]

Examples:
registry.access.redhat.com/ubi8/python-39:latest
│                            │    │         │
│                            │    │         └─ tag (version)
│                            │    └─────────── image name
│                            └──────────────── namespace
└───────────────────────────────────────────── registry host
```

**Semantic Versioning for container images:**
```
MAJOR.MINOR.PATCH

Example: 2.5.3
         │ │ └── 3 = bug fixes, security patches
         │ └──── 5 = new features, backwards compatible
         └────── 2 = breaking changes
```

**The :latest tag problem — a critical concept:**

> **From the book:** "Using the latest tag is considered a bad practice. Because the latest tag also represents the latest version of the image, it can include backwards-incompatible changes and cause containers that use the image to break."

```bash
# BAD practice — latest can change without warning
podman pull quay.io/argoproj/argocd        # uses :latest implicitly
podman pull quay.io/argoproj/argocd:latest # same thing, explicit

# GOOD practice — pin to specific version
podman pull quay.io/argoproj/argocd:v2.8.4

# BEST practice for production — pin to digest (immutable)
podman pull quay.io/argoproj/argocd@sha256:abc123...
```

**What senior engineers do:**
- Dev/Test environments: use semantic version tags (`v2.8.4`)
- Production: pin to **digest** (`@sha256:...`) — immutable, never changes
- CI/CD pipelines: build with version tag + SHA, push both
- Never use `:latest` in production Kubernetes/OpenShift manifests

---

### Searching for Images

```bash
# Search across ALL registries in registries.conf
podman search nginx
# NAME                                          DESCRIPTION
# registry.fedoraproject.org/f29/nginx
# registry.access.redhat.com/ubi8/nginx-118    Platform for running nginx 1.18...
# docker.io/library/nginx                      Official build of Nginx.
# quay.io/linuxserver.io/baseimage-alpine-nginx

# Search for Python images
podman search python

# Search with format filter
podman search --format "table {{.Name}}\t{{.Stars}}\t{{.Official}}" nginx

# Search on specific registry only
podman search registry.access.redhat.com/httpd

# Search with no truncation of description
podman search --no-trunc registry.access.redhat.com/python
```

**Alternative — use the web UI:**
- Red Hat images: https://catalog.redhat.com/software/containers/explore
- Quay.io: https://quay.io/search
- Docker Hub: https://hub.docker.com

---

### Pulling Images

```bash
# Syntax
podman pull IMAGE_NAME
podman image pull IMAGE_NAME   # same thing, longer form

# Pull with full qualified name (recommended)
podman pull registry.redhat.io/ubi8/ubi:8.6

# Pull without registry (uses registries.conf search order)
podman pull ubi8/python-39

# Pull specific tag
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman pull registry.redhat.io/rhel8/mariadb-103:1

# Pull from Quay.io
podman pull quay.io/your-user/your-image:tag

# Pull by digest (immutable — production best practice)
podman pull registry.access.redhat.com/ubi8@sha256:abc123...
```

**Pull output explained:**
```
Trying to pull registry.redhat.io/ubi8/ubi:8.6...   ← connects to registry
Getting image source signatures                        ← verifies image signature
Checking if image destination supports signatures
Copying blob 7697bc3a2b4b done                        ← downloads each layer blob
Copying blob a4e8f99c2c95 done
Writing manifest to image destination                 ← saves manifest (index)
Storing signatures                                    ← stores image verification
3434...8f6b                                           ← short SHA ID of the image
```

**Where images are stored:**
```bash
# Rootless (normal user) — images stored here:
~/.local/share/containers/storage/

# Root user — images stored here:
/var/lib/containers/storage/

# List your local images
podman image ls
podman images   # same thing, shorter form

# Output:
# REPOSITORY                                TAG    IMAGE ID     CREATED     SIZE
# registry.redhat.io/rhel9/python-39        1-52   d336a3191d35 3 weeks ago 995 MB
# registry.access.redhat.com/ubi8/python-39 latest 6b7a42c9d513 4 weeks ago 894 MB
```

> **Exam tip:** Root-pulled images are NOT listed when running `podman image ls` as a regular user. Root images are in `/var/lib/containers`, user images in `~/.local/share/containers`.

---

### Building Images

```bash
# Syntax
podman build --file CONTAINERFILE --tag IMAGE_REFERENCE

# Build and tag for Quay.io
podman build --file Containerfile \
  --tag quay.io/YOUR_QUAY_USER/IMAGE_NAME:TAG

# Shorter form (-f and -t)
podman build -f Containerfile -t quay.io/myuser/myapp:v1.0.0

# Build from current directory (Containerfile must be named "Containerfile" or "Dockerfile")
podman build -t myapp:latest .

# Build with build arguments
podman build --build-arg VERSION=2.0 -t myapp:2.0 .
```

**Example Containerfile (from the book's lab):**
```dockerfile
FROM registry.access.redhat.com/ubi9/nginx-120:1-39
RUN echo "It is pitch black. You are likely to be eaten by a grue." >> index.html
CMD nginx -g "daemon off;"
```

---

### Pushing Images

```bash
# Must be logged in BEFORE pushing
podman login quay.io

# Push to Quay.io
podman push quay.io/YOUR_QUAY_USER/IMAGE_NAME:TAG

# Push output:
# Getting image source signatures
# Copying blob fb3154998920 done     ← each layer uploaded
# Copying blob a12c456789ab done
# Writing manifest to image destination
# Storing signatures
```

**Full build-tag-push workflow:**
```bash
# Step 1: Build locally
podman build -t myapp:v1.0.0 .

# Step 2: Tag for registry
podman tag myapp:v1.0.0 quay.io/myuser/myapp:v1.0.0
podman tag myapp:v1.0.0 quay.io/myuser/myapp:latest

# Step 3: Login
podman login quay.io

# Step 4: Push both tags
podman push quay.io/myuser/myapp:v1.0.0
podman push quay.io/myuser/myapp:latest
```

---

### Inspecting Images

```bash
# Full JSON inspection
podman image inspect registry.redhat.io/rhel8/mariadb-103:1
podman inspect registry.redhat.io/rhel8/mariadb-103:1   # same

# Key fields in the inspection output:
# - Id: image SHA256 hash
# - Config.User: default user the container runs as
# - Config.ExposedPorts: ports the application uses
# - Config.Env: environment variables baked into the image
# - Config.Entrypoint: the main executable
# - Config.Cmd: the default command / arguments
# - Config.WorkingDir: working directory for commands
# - Config.Labels: metadata labels
# - Architecture: cpu architecture (amd64, arm64)
# - Os: operating system
# - Size: image size in bytes

# Extract specific field with Go template
podman image inspect registry.redhat.io/rhel8/mariadb-103:1 \
  --format="{{.Config.Cmd}}"
# Output: [run-mysqld]

# Other useful format queries
podman inspect myimage --format "{{.Config.User}}"         # default user
podman inspect myimage --format "{{.Config.ExposedPorts}}" # ports
podman inspect myimage --format "{{.Config.Env}}"          # env vars
podman inspect myimage --format "{{.Config.Entrypoint}}"   # entrypoint
podman inspect myimage --format "{{.Config.WorkingDir}}"   # workdir
podman inspect myimage --format "{{.Config.Labels}}"       # labels
podman inspect myimage --format "{{.Architecture}}"        # cpu arch
podman inspect myimage --format "{{.Os}}"                  # OS
podman inspect myimage --format "{{.Size}}"                # size in bytes
podman inspect myimage --format "{{.Digest}}"              # SHA256 digest

# View image layer history
podman history myimage
podman history --no-trunc myimage  # show full commands

# Inspect REMOTE image without pulling (using Skopeo)
skopeo inspect docker://registry.access.redhat.com/ubi8/python-39:latest
skopeo list-tags docker://registry.access.redhat.com/ubi8
```

> **Book note on Skopeo:** "To inspect a remote image, you can use the Skopeo tool."

---

### Tagging Images

```bash
# Syntax
podman image tag LOCAL_IMAGE:TAG LOCAL_IMAGE:NEW_TAG
podman tag LOCAL_IMAGE:TAG LOCAL_IMAGE:NEW_TAG   # same, shorter

# Add a new tag to a local image
podman image tag myapp:latest myapp:v2.5.0
podman image tag myapp:latest myapp:stable

# Tag for pushing to a registry
podman tag myapp:v1.0.0 quay.io/myuser/myapp:v1.0.0

# Tag with new name (from the book's lab)
podman tag images-lab images-lab:grue

# Multiple tags for same image
podman tag ubi8:latest mybase:latest
podman tag ubi8:latest mybase:8
podman tag ubi8:latest mybase:rhel8
# All three tags point to the same image ID
```

**Key insight:** Tags are just pointers to the same image data. Multiple tags can point to the same Image ID. Removing a tag does NOT delete the image unless it was the last tag.

---

### Removing Images

```bash
# Remove a single image
podman image rm REGISTRY/NAMESPACE/IMAGE_NAME:TAG
podman rmi REGISTRY/NAMESPACE/IMAGE_NAME:TAG   # same

# Remove by image ID (short or long)
podman rmi d336a3191d35

# Force remove (stops and removes any containers using it first)
podman image rm -f REGISTRY/NAMESPACE/IMAGE_NAME:TAG

# Remove ALL local images
podman rmi --all
podman image rm --all

# Dangling images
# = images with no tag AND not referenced by another image
# Created when you rebuild an image with the same tag
podman image prune              # remove ONLY dangling images
# WARNING! This command removes all dangling images.
# Are you sure you want to continue? [y/N]

# Remove dangling + ALL unused images
podman image prune -a
# WARNING! This command removes all images without at least one container associated with them.
# Are you sure you want to continue? [y/N]

# Skip confirmation prompt
podman image prune -af          # dangling + unused, no prompt
podman image prune -f           # only dangling, no prompt

# Check disk usage before and after
podman system df
```

**When podman rmi fails:**
```
Error: Image used by container XYZ (cannot remove)

Solution:
podman stop container-name       # stop the container first
podman rm container-name         # remove the container
podman rmi image-name            # now remove the image

OR:
podman rmi -f image-name         # force: stops + removes containers then image
```

---

## Complete Command Reference

### All Chapter 3 Commands

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `podman login REGISTRY` | Authenticate to a registry | — |
| `podman logout REGISTRY` | Log out of a registry | — |
| `podman search TERM` | Search for images | `--format`, `--no-trunc` |
| `podman pull IMAGE` | Download image to local storage | — |
| `podman image pull IMAGE` | Same as above (longer form) | — |
| `podman images` | List local images | `--format`, `--filter` |
| `podman image ls` | Same as above (longer form) | — |
| `podman image inspect IMAGE` | Full metadata as JSON | `--format` |
| `podman inspect IMAGE` | Same as above | `--format` |
| `podman history IMAGE` | Show layer history | `--no-trunc` |
| `podman build -f FILE -t TAG` | Build image from Containerfile | `--build-arg` |
| `podman push IMAGE` | Upload image to registry | — |
| `podman tag SRC DEST` | Create a new tag | — |
| `podman image tag SRC DEST` | Same as above (longer form) | — |
| `podman rmi IMAGE` | Remove a local image | `-f`, `--all` |
| `podman image rm IMAGE` | Same as above (longer form) | `-f`, `--all` |
| `podman image prune` | Remove dangling images | `-a`, `-f` |
| `podman system df` | Show disk usage | — |
| `skopeo inspect docker://IMAGE` | Inspect remote image (no pull) | — |
| `skopeo list-tags docker://REPO` | List available tags | — |

---

## Labs — Basic (per topic)

### Lab B1: Registry exploration — Red Hat Catalog

```bash
# 1. Navigate to Red Hat Catalog
# https://catalog.redhat.com/software/containers/explore

# 2. Search for "ubi8/python"
# Click on ubi8/python-38

# 3. Explore each tab:
# Overview  → description and usage info
# Security  → Health Index (A-F), CVE details
# Technical → ports, architecture, working directory
# Packages  → all installed RPM packages (hundreds!)
# Dockerfile → exact Containerfile that built this image

# 4. Get the pull command from "Get this image" tab

# 5. Pull it locally
podman pull registry.access.redhat.com/ubi8/python-38:latest

# 6. Verify
podman images | grep python-38
```

---

### Lab B2: Login and authentication

```bash
# 1. Check if already logged in
podman login --get-login registry.redhat.io 2>/dev/null || echo "Not logged in"

# 2. Login to registry.access.redhat.com (free — no account needed)
# No login required for this registry!
podman pull registry.access.redhat.com/ubi8:latest

# 3. Login to registry.redhat.io (needs Red Hat account)
podman login registry.redhat.io
# Enter your credentials

# 4. View stored credentials
cat ${XDG_RUNTIME_DIR}/containers/auth.json

# 5. Decode the base64 credential
AUTH=$(cat ${XDG_RUNTIME_DIR}/containers/auth.json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['auths']['registry.redhat.io']['auth'])")
echo -n "$AUTH" | base64 -d

# 6. Login to Quay.io
podman login quay.io

# 7. Logout from all registries
podman logout registry.redhat.io
podman logout quay.io
```

---

### Lab B3: Search for images

```bash
# 1. Search for nginx across all configured registries
podman search nginx

# 2. Format output as table
podman search --format "table {{.Name}}\t{{.Stars}}\t{{.Official}}" nginx

# 3. Search only on Red Hat registry
podman search registry.access.redhat.com/httpd

# 4. Search for Python
podman search python

# 5. Search with no truncation
podman search --no-trunc registry.access.redhat.com/ubi8

# 6. Use skopeo to list tags without pulling
skopeo list-tags docker://registry.access.redhat.com/ubi8

# 7. Inspect a remote image without pulling
skopeo inspect docker://registry.access.redhat.com/ubi8/python-39:latest
```

---

### Lab B4: Pull images and explore local storage

```bash
# 1. Pull UBI8 minimal image
podman pull registry.access.redhat.com/ubi8/ubi-minimal:8.5

# 2. Pull Python image
podman pull registry.access.redhat.com/ubi8/python-39:latest

# 3. List all local images
podman images
podman image ls

# 4. Show with full format
podman images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}"

# 5. Show storage location
ls ~/.local/share/containers/storage/
# (rootless user image storage)

# 6. Filter images by name
podman images --filter reference="*ubi8*"

# 7. Show only image IDs
podman images -q

# 8. Show image sizes
podman images --format "{{.Repository}}:{{.Tag}} = {{.Size}}"
```

---

### Lab B5: Inspect images

```bash
# 1. Pull an image to inspect
podman pull registry.access.redhat.com/ubi8/python-39:latest

# 2. Full JSON inspect
podman image inspect registry.access.redhat.com/ubi8/python-39:latest

# 3. Extract specific fields with Go templates
podman inspect ubi8/python-39:latest --format "{{.Config.Cmd}}"
podman inspect ubi8/python-39:latest --format "{{.Config.User}}"
podman inspect ubi8/python-39:latest --format "{{.Config.Env}}"
podman inspect ubi8/python-39:latest --format "{{.Config.ExposedPorts}}"
podman inspect ubi8/python-39:latest --format "{{.Config.WorkingDir}}"
podman inspect ubi8/python-39:latest --format "{{.Architecture}}"
podman inspect ubi8/python-39:latest --format "{{.Os}}"
podman inspect ubi8/python-39:latest --format "{{.Size}}"
podman inspect ubi8/python-39:latest --format "{{.Digest}}"

# 4. View image layer history
podman history registry.access.redhat.com/ubi8/python-39:latest
podman history --no-trunc registry.access.redhat.com/ubi8/python-39:latest

# 5. Count the layers
podman history ubi8/python-39:latest | wc -l
```

---

### Lab B6: Tagging images

```bash
# 1. Pull a base image
podman pull registry.access.redhat.com/ubi8:latest

# 2. Add multiple tags to the same image
podman tag registry.access.redhat.com/ubi8:latest mybase:latest
podman tag registry.access.redhat.com/ubi8:latest mybase:8
podman tag registry.access.redhat.com/ubi8:latest mybase:rhel8
podman tag registry.access.redhat.com/ubi8:latest mybase:stable

# 3. Verify all tags point to same image ID
podman images | grep mybase
# All rows show the same IMAGE ID

# 4. Tag for pushing to Quay
podman tag mybase:latest quay.io/YOUR_USER/mybase:latest
podman tag mybase:latest quay.io/YOUR_USER/mybase:v1.0.0
podman images | grep YOUR_USER

# 5. Remove one tag (image stays — other tags still exist)
podman rmi mybase:stable
podman images | grep mybase   # mybase:latest, :8, :rhel8 still exist

# 6. Cleanup
podman rmi mybase:latest mybase:8 mybase:rhel8
podman rmi quay.io/YOUR_USER/mybase:latest quay.io/YOUR_USER/mybase:v1.0.0
```

---

### Lab B7: Remove images and prune storage

```bash
# 1. Setup — pull several images
podman pull registry.access.redhat.com/ubi8:latest
podman pull registry.access.redhat.com/ubi8-minimal:latest
podman pull registry.access.redhat.com/ubi8/python-39:latest

# 2. Check storage before cleanup
podman system df
podman images

# 3. Remove a single image
podman rmi registry.access.redhat.com/ubi8-minimal:latest
podman images   # confirm it's gone

# 4. Try to remove image in use by a container
podman run -d --name webtest registry.access.redhat.com/ubi8/python-39:latest sleep 3600
podman rmi registry.access.redhat.com/ubi8/python-39:latest
# Error: Image is in use by container webtest

# 5. Force remove (kills container too)
podman rmi -f registry.access.redhat.com/ubi8/python-39:latest
podman ps -a   # container also removed!

# 6. Create dangling images
podman build -t danglingtest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "version 1"
EOF
# Rebuild with same tag (old image becomes dangling)
podman build -t danglingtest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "version 2"
EOF
podman images --filter dangling=true   # see the dangling image

# 7. Prune dangling images
podman image prune -f

# 8. Prune ALL unused images
podman image prune -af

# 9. Final cleanup
podman stop -a && podman rm -a
podman system prune -af
podman system df   # confirm clean
```

---

## Labs — Scenario Based

### Lab S1: Pull from Quay, tag, and run (book lab exercise)

**Scenario:** This is the actual DO188 Chapter 3 Lab Exercise from the student guide.

```bash
# Step 1: Login to Quay.io
podman login quay.io
# Enter your Quay.io username and password

# Step 2: Build the image on Quay.io (via web UI)
# Go to https://quay.io → Create New Repository
# Repository Name: images-lab
# Repository Visibility: Public
# Initialize repository: Initialize from a Dockerfile
# Use this Containerfile:
cat << 'EOF'
FROM registry.access.redhat.com/ubi9/nginx-120:1-39
RUN echo "It is pitch black. You are likely to be eaten by a grue." >> index.html
CMD nginx -g "daemon off;"
EOF

# Step 3: Pull the built image from Quay
podman pull quay.io/YOUR_QUAY_USER/images-lab
# Trying to pull quay.io/username/images-lab:latest...
# ...
# b5e0...d858

# Step 4: Add the images-lab:grue tag
podman tag images-lab images-lab:grue
# No output expected

# Step 5: Verify both tags exist
podman images | grep images-lab
# images-lab   latest  <same ID>
# images-lab   grue    <same ID>

# Step 6: Run the container using the grue tag
podman run -d \
  --name images-lab \
  -p 8080:8080 \
  images-lab:grue

# Step 7: Verify (optional)
curl localhost:8080
# It is pitch black. You are likely to be eaten by a grue.

# Cleanup
podman stop images-lab && podman rm images-lab
podman rmi images-lab:grue images-lab:latest
```

---

### Lab S2: Guided exercise — Container Image Registries

**Scenario:** This recreates the DO188 Guided Exercise: images-basics.

```bash
# Step 1: Create basic image on Quay (via web UI)
# Go to https://quay.io → Create New Repository
# Repository Name: do188-basic-image
# Repository Visibility: Public
# Initialize from Dockerfile using this Containerfile:
cat << 'EOF'
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.5
CMD echo "Hello from the container"
EOF

# Step 2: Test the image — run from Quay
podman run quay.io/YOUR_QUAY_USER/do188-basic-image
# Trying to pull quay.io/YOUR_QUAY_USER/do188-test:latest...
# Hello from the container

# Step 3: Explore image in Red Hat Catalog
# Go to: https://catalog.redhat.com/software/containers/search
# Search: ubi8/python
# Click: ubi8/python-38
# Explore: Overview, Security, Technical Information, Packages, Dockerfile, Get this image

# Step 4: Pull the Python UBI image from catalog
podman pull registry.access.redhat.com/ubi8/python-38:1-96

# Step 5: Run it and check Python version
podman run --name python \
  registry.access.redhat.com/ubi8/python-38:1-96 \
  python3 --version
# Python 3.8.12

# Cleanup
podman rm python
podman rmi registry.access.redhat.com/ubi8/python-38:1-96
```

---

### Lab S3: Guided exercise — Managing images

**Scenario:** Recreating the DO188 Guided Exercise: images-managing.

```bash
# Step 1: Search for Python image
podman search python
# Look for: registry.access.redhat.com/ubi8/python-39

# Step 2: Pull the image
podman image pull registry.access.redhat.com/ubi8/python-39:latest
# Trying to pull ...
# Getting image source signatures
# ...

# Step 3: Run a Python HTTP server from the pulled image
podman run \
  --rm \
  -p 8080:8000 \
  --name http-server-pulled \
  ubi8/python-39 python -m http.server
# Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/)

# Step 4: Test from another terminal (or browser)
curl http://localhost:8080
# <!DOCTYPE HTML PUBLIC ...>

# Step 5: Stop with Ctrl+C

# Step 6: Clean up all unused images
podman image prune -af

# Step 7: Verify cleanup
podman images
```

---

### Lab S4: Multi-registry workflow (real enterprise scenario)

**Scenario:** You are a DevOps engineer. Pull a Red Hat certified image, inspect it for security, tag it for your company's internal registry, and push it.

```bash
# Step 1: Login to Red Hat registry
podman login registry.redhat.io

# Step 2: Pull the mariadb image from Red Hat
podman pull registry.redhat.io/rhel8/mariadb-103:1

# Step 3: Security inspection — check what user it runs as
podman inspect registry.redhat.io/rhel8/mariadb-103:1 \
  --format "Default User: {{.Config.User}}"
# Default User: 27

# Step 4: Check exposed ports
podman inspect registry.redhat.io/rhel8/mariadb-103:1 \
  --format "Ports: {{.Config.ExposedPorts}}"
# Ports: map[3306/tcp:{}]

# Step 5: Check the default command
podman inspect registry.redhat.io/rhel8/mariadb-103:1 \
  --format "CMD: {{.Config.Cmd}}"
# CMD: [run-mysqld]

# Step 6: Check the entrypoint
podman inspect registry.redhat.io/rhel8/mariadb-103:1 \
  --format "Entrypoint: {{.Config.Entrypoint}}"
# Entrypoint: [container-entrypoint]

# Step 7: Tag for internal company registry
INTERNAL_REGISTRY=registry.company.internal
podman tag registry.redhat.io/rhel8/mariadb-103:1 \
  ${INTERNAL_REGISTRY}/databases/mariadb:10.3-rhel8

# Step 8: (If registry is available) Login and push
# podman login ${INTERNAL_REGISTRY}
# podman push ${INTERNAL_REGISTRY}/databases/mariadb:10.3-rhel8

# Step 9: Cleanup
podman rmi registry.redhat.io/rhel8/mariadb-103:1
```

---

### Lab S5: Configure registries.conf for a team environment

**Scenario:** Your team uses an internal registry. Configure Podman to search it first, and block Docker Hub for security compliance.

```bash
# Step 1: View current registry configuration
cat /etc/containers/registries.conf

# Step 2: Create user-level override (no root required)
mkdir -p ~/.config/containers/

cat > ~/.config/containers/registries.conf << 'EOF'
# Team registry configuration
unqualified-search-registries = [
  "registry.company.internal",
  "registry.access.redhat.com",
  "quay.io"
]

# Block Docker Hub (security policy)
[[registry]]
location = "docker.io"
blocked = true

# Allow Red Hat registries
[[registry]]
location = "registry.access.redhat.com"
blocked = false

[[registry]]
location = "registry.redhat.io"
blocked = false
EOF

# Step 3: Test search uses new configuration
podman search python
# Should search company registry first, then access.redhat.com, then quay.io
# Should NOT show docker.io results

# Step 4: Verify docker.io is blocked
podman pull docker.io/nginx:latest 2>&1 | grep -i blocked

# Step 5: Reset to default (remove user-level config)
rm ~/.config/containers/registries.conf
```

---

## Labs — Intermediate

### Lab I1: Full image lifecycle — build, tag, push, pull, run, remove

```bash
# Setup: Create a simple application
mkdir ~/ch3-lifecycle && cd ~/ch3-lifecycle

cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>DO188 Chapter 3 Lab</title></head>
<body>
  <h1>Ahmad Ali — DO188 Chapter 3 Complete!</h1>
  <p>Container Images lab — built and pushed to Quay.io</p>
</body>
</html>
EOF

cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi9/nginx-120:1-39
COPY index.html /usr/share/nginx/html/index.html
CMD nginx -g "daemon off;"
EOF

# Step 1: Build the image
podman build -f Containerfile -t mywebapp:v1.0.0 .
podman images | grep mywebapp

# Step 2: Test locally before pushing
podman run -d --name webtest -p 8080:8080 mywebapp:v1.0.0
curl http://localhost:8080
podman stop webtest && podman rm webtest

# Step 3: Tag for Quay.io
podman login quay.io
podman tag mywebapp:v1.0.0 quay.io/YOUR_USER/mywebapp:v1.0.0
podman tag mywebapp:v1.0.0 quay.io/YOUR_USER/mywebapp:latest

# Step 4: Push to registry
podman push quay.io/YOUR_USER/mywebapp:v1.0.0
podman push quay.io/YOUR_USER/mywebapp:latest

# Step 5: Remove local image
podman rmi mywebapp:v1.0.0
podman rmi quay.io/YOUR_USER/mywebapp:v1.0.0
podman rmi quay.io/YOUR_USER/mywebapp:latest

# Step 6: Pull from registry (proves it works end-to-end)
podman pull quay.io/YOUR_USER/mywebapp:v1.0.0
podman run -d --name fromquay -p 8080:8080 quay.io/YOUR_USER/mywebapp:v1.0.0
curl http://localhost:8080

# Cleanup
podman stop fromquay && podman rm fromquay
podman rmi quay.io/YOUR_USER/mywebapp:v1.0.0
rm -rf ~/ch3-lifecycle
```

---

### Lab I2: Image inspection mastery

```bash
# Pull a complex image with lots of metadata
podman pull registry.access.redhat.com/ubi8/python-39:latest

# 1. Full inspection (browse the JSON structure)
podman image inspect ubi8/python-39:latest | python3 -m json.tool | less

# 2. Go template: all env vars, one per line
podman inspect ubi8/python-39:latest \
  --format '{{range .Config.Env}}{{println .}}{{end}}'

# 3. Go template: all labels
podman inspect ubi8/python-39:latest \
  --format '{{range $k, $v := .Config.Labels}}{{$k}}={{$v}}{{println}}{{end}}'

# 4. Image size in megabytes
podman inspect ubi8/python-39:latest \
  --format '{{printf "%.2f MB" (div .Size 1048576)}}'

# 5. Full image digest
podman inspect ubi8/python-39:latest --format '{{.Digest}}'

# 6. Number of layers
podman inspect ubi8/python-39:latest --format '{{len .RootFS.Layers}}'

# 7. Creation date
podman inspect ubi8/python-39:latest --format '{{.Created}}'

# 8. History — see each layer command
podman history ubi8/python-39:latest

# Cleanup
podman rmi ubi8/python-39:latest
```

---

### Lab I3: Semantic versioning workflow

```bash
# Simulate a proper image versioning workflow
mkdir ~/semver-lab && cd ~/semver-lab

# Version 1.0.0 release
cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
LABEL version="1.0.0"
LABEL description="My app"
CMD echo "App version 1.0.0"
EOF

podman build -t myapp:1.0.0 .
podman tag myapp:1.0.0 myapp:1
podman tag myapp:1.0.0 myapp:latest

# Version 1.1.0 release (minor — new features, compatible)
cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
LABEL version="1.1.0"
LABEL description="My app - added feature X"
CMD echo "App version 1.1.0 - with feature X"
EOF

podman build -t myapp:1.1.0 .
podman tag myapp:1.1.0 myapp:1
podman tag myapp:1.1.0 myapp:latest

# Version 1.1.1 release (patch — bug fix)
cat > Containerfile << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
LABEL version="1.1.1"
LABEL description="My app - bugfix"
CMD echo "App version 1.1.1 - with bugfix"
EOF

podman build -t myapp:1.1.1 .
podman tag myapp:1.1.1 myapp:1.1
podman tag myapp:1.1.1 myapp:1
podman tag myapp:1.1.1 myapp:latest

# List all versions
podman images | grep myapp

# Prove latest != pinned version
podman run --rm myapp:1.0.0    # shows 1.0.0
podman run --rm myapp:1.1.1    # shows 1.1.1
podman run --rm myapp:latest   # shows 1.1.1 (could change tomorrow!)

# Cleanup
podman rmi -f myapp:1.0.0 myapp:1.1.0 myapp:1.1.1 myapp:1.1 myapp:1 myapp:latest
rm -rf ~/semver-lab
```

---

### Lab I4: Image pruning and storage management

```bash
# Setup: Create a scenario with multiple unused images
podman pull registry.access.redhat.com/ubi8:latest
podman pull registry.access.redhat.com/ubi8-minimal:latest
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman pull registry.access.redhat.com/ubi8/nodejs-16:latest

# Check disk usage
podman system df
echo "---"
podman images

# Simulate dangling images (rebuild same tag)
podman build -t temptest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "build 1"
EOF

podman build -t temptest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "build 2"
EOF
# First build's image is now dangling

# See dangling images
echo "=== Dangling images ==="
podman images --filter dangling=true

# Prune ONLY dangling (safe)
podman image prune -f
echo "=== After pruning dangling ==="
podman images

# Create a container using an image (to protect it from prune -a)
podman run -d --name protectedapp registry.access.redhat.com/ubi8/python-39:latest sleep 3600

# Prune unused but leave images used by containers
podman image prune -af
echo "=== After prune -af ==="
podman images
# python-39 stays! (protected by running container)
# ubi8, ubi8-minimal, nodejs-16 are REMOVED (not used by any container)

# Cleanup everything
podman stop protectedapp && podman rm protectedapp
podman image prune -af
podman system df
```

---

### Lab I5: Compare UBI variants

```bash
# Pull all UBI variants
podman pull registry.access.redhat.com/ubi8:latest
podman pull registry.access.redhat.com/ubi8/ubi-minimal:latest
podman pull registry.access.redhat.com/ubi8/ubi-micro:latest 2>/dev/null || \
  podman pull registry.access.redhat.com/ubi8-micro:latest 2>/dev/null || true

# Compare sizes
echo "=== Image Sizes ==="
podman images --format "{{.Repository}}:{{.Tag}} — {{.Size}}" | grep ubi

# Compare available tools
echo "=== Full UBI8 ==="
podman run --rm ubi8:latest which dnf bash rpm curl 2>&1
podman run --rm ubi8:latest rpm -qa | wc -l

echo "=== UBI8 Minimal ==="
podman run --rm ubi8/ubi-minimal:latest which microdnf 2>&1
podman run --rm ubi8/ubi-minimal:latest rpm -qa | wc -l

echo "=== UBI8 Micro ==="
podman run --rm ubi8-micro:latest ls /bin 2>/dev/null | head -10 || echo "Very minimal!"

# Compare architecture
for img in ubi8:latest ubi8/ubi-minimal:latest; do
  echo "$img: $(podman inspect $img --format '{{.Architecture}}')"
done

# Cleanup
podman rmi ubi8:latest ubi8/ubi-minimal:latest ubi8-micro:latest 2>/dev/null; true
```

---

## Labs — Industry / Interview Questions with Solutions

### Q1: What is the difference between registry.access.redhat.com and registry.redhat.io?

**Answer:**
- `registry.access.redhat.com` — Free, public, **no authentication required**. Contains UBI images only.
- `registry.redhat.io` — Requires **Red Hat account login**. Contains full RHEL product images, requires a subscription for full support.

```bash
# Proof — pull from access.redhat.com (no login needed)
podman pull registry.access.redhat.com/ubi8:latest   # works without login

# Proof — pull from registry.redhat.io (login required)
podman logout registry.redhat.io 2>/dev/null
podman pull registry.redhat.io/rhel8/httpd-24 2>&1 | grep -i "error\|unauthorized\|login"
# Error: unauthorized: access denied
```

---

### Q2: What is a dangling image and how do you remove it?

**Answer:** A dangling image is an image that has **no tag** AND is **not referenced by any other image**. They are created when you rebuild an image using the same tag — the old image loses its tag, becoming dangling.

```bash
# Create a dangling image
podman build -t dangletest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "build 1"
EOF

podman build -t dangletest:latest - << 'EOF'
FROM registry.access.redhat.com/ubi8:latest
CMD echo "build 2"
EOF

# See the dangling image
podman images --filter dangling=true
# REPOSITORY  TAG      IMAGE ID     CREATED    SIZE
# <none>      <none>   abc123...    1 min ago  214MB  ← DANGLING

# Remove all dangling images
podman image prune -f

# Remove dangling + all unused images
podman image prune -af
```

---

### Q3: Why is using the :latest tag a bad practice?

**Answer:** The `:latest` tag is mutable — it points to whatever the most recent image is. If the image maintainer pushes a breaking change and tags it `:latest`, your container breaks on next pull. You have no guarantee of consistency.

```bash
# Demonstrate the problem
# Week 1: you pull nginx:latest — gets version 1.24
podman pull docker.io/nginx:latest
podman inspect nginx:latest --format "{{.Config.Labels}}"

# Week 2: maintainer releases nginx 1.25 with :latest tag
# Now: podman pull nginx:latest gets 1.25 — potentially breaks your app!

# Solution 1: pin to specific version
podman pull registry.access.redhat.com/ubi8/python-39:1-52

# Solution 2 (production): pin to digest (IMMUTABLE — never changes)
DIGEST=$(podman inspect ubi8/python-39:latest --format "{{.Digest}}")
podman pull registry.access.redhat.com/ubi8/python-39@${DIGEST}
echo "Pinned digest: ${DIGEST}"
# sha256:abc123... — this NEVER changes, ALWAYS same image
```

---

### Q4: How do you find what command runs by default in a container image?

**Answer:** Use `podman inspect` with Go template to extract `Config.Cmd` and `Config.Entrypoint`.

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest

# Get the default CMD
podman inspect ubi8/python-39:latest --format "{{.Config.Cmd}}"
# Example: [/bin/sh -c $STI_SCRIPTS_PATH/run]

# Get the ENTRYPOINT
podman inspect ubi8/python-39:latest --format "{{.Config.Entrypoint}}"
# Example: [container-entrypoint]

# Get BOTH together
podman inspect ubi8/python-39:latest \
  --format "Entrypoint={{.Config.Entrypoint}} CMD={{.Config.Cmd}}"

# Alternative: view history to see what each layer added
podman history ubi8/python-39:latest --format "{{.CreatedBy}}"
```

---

### Q5: How does Podman find an image when no registry is specified?

**Answer:** Podman uses `/etc/containers/registries.conf` to find the `unqualified-search-registries` list, then searches each registry in order until the image is found.

```bash
# Demonstrate unqualified name resolution
cat /etc/containers/registries.conf | grep unqualified

# Example config: ['registry.redhat.io', 'docker.io']
# So: podman pull python-39
# → tries registry.redhat.io/python-39 first
# → if not found, tries docker.io/python-39

# This is WHY qualified names are safer:
podman pull ubi8/python-39           # ambiguous — depends on registries.conf
podman pull registry.access.redhat.com/ubi8/python-39:latest  # explicit and safe

# Security risk: someone could push a malicious image to a searched registry
# with the same name as a legitimate image. Always use qualified names!
```

---

### Q6: Where does Podman store credentials after podman login?

**Answer:** In `${XDG_RUNTIME_DIR}/containers/auth.json`. The password is stored BASE64 encoded (not encrypted — base64 is just encoding).

```bash
podman login registry.access.redhat.com  # (no password needed for this one)

# See where credentials file is
echo "Credential file: ${XDG_RUNTIME_DIR}/containers/auth.json"
cat ${XDG_RUNTIME_DIR}/containers/auth.json

# Decode the stored credential
AUTH_VALUE="dXNlcjpodW50ZXIy"   # example base64 value
echo -n "${AUTH_VALUE}" | base64 -d
# Output: user:hunter2

# Security implication: anyone who can read this file gets your password!
# Enterprise solution: use credential helpers (docker-credential-secretservice, etc.)
# Or: use short-lived tokens, not passwords
```

---

### Q7: What is the difference between podman image prune and podman image prune -a?

**Answer:**
- `podman image prune` — removes only **dangling** images (untagged, unreferenced)
- `podman image prune -a` — removes dangling images PLUS all **unused** images (no running/stopped container uses them)

```bash
# Setup
podman pull registry.access.redhat.com/ubi8:latest
podman pull registry.access.redhat.com/ubi8-minimal:latest

# Create a running container (protects ubi8:latest from prune -a)
podman run -d --name keeper registry.access.redhat.com/ubi8:latest sleep 3600

# prune WITHOUT -a: only removes dangling (untagged) images
podman image prune -f
podman images   # ubi8 and ubi8-minimal still present

# prune WITH -a: removes all images not used by any container
podman image prune -af
podman images   # ubi8:latest stays (keeper container uses it)
               # ubi8-minimal is GONE (no container uses it)

# Cleanup
podman stop keeper && podman rm keeper
podman image prune -af
```

---

### Q8: How do you add multiple tags to the same image?

**Answer:** Run `podman tag` multiple times with the same source image. All tags point to the same image ID.

```bash
podman pull registry.access.redhat.com/ubi8:latest

# Add multiple tags
podman tag ubi8:latest myapp:2.5.0
podman tag ubi8:latest myapp:2.5
podman tag ubi8:latest myapp:2
podman tag ubi8:latest myapp:latest
podman tag ubi8:latest myapp:stable
podman tag ubi8:latest myapp:production

# Verify all share same IMAGE ID
podman images | grep myapp
# All show identical IMAGE ID

# Remove one tag (others remain)
podman rmi myapp:stable
podman images | grep myapp   # production, latest, 2, 2.5, 2.5.0 all still there

# Remove the image (must remove ALL tags, or use --all)
podman rmi myapp:2.5.0 myapp:2.5 myapp:2 myapp:latest myapp:production
# OR:
podman rmi -f myapp  # removes all tags and the image
```

---

### Q9: How do you inspect an image that is stored on a remote registry WITHOUT pulling it?

**Answer:** Use `skopeo inspect` — it reads image metadata directly from the registry.

```bash
# Inspect remote image without pulling
skopeo inspect docker://registry.access.redhat.com/ubi8/python-39:latest

# List available tags for an image without pulling
skopeo list-tags docker://registry.access.redhat.com/ubi8

# Get specific field from remote image
skopeo inspect docker://registry.access.redhat.com/ubi8/python-39:latest \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print('Architecture:', d.get('Architecture'))"

# Compare remote vs local
skopeo inspect docker://registry.access.redhat.com/ubi8:latest | python3 -c \
  "import sys,json; d=json.load(sys.stdin); print('Digest:', d.get('Digest'))"
podman inspect ubi8:latest --format "{{.Digest}}"
# If they match → your local image is up to date
# If they differ → a newer version exists on the registry
```

---

### Q10: What happens if you run podman rmi on an image with a running container?

**Answer:** Podman refuses and returns an error unless you use `-f` (force). Force stops the container, removes it, then removes the image.

```bash
# Setup
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman run -d --name testapp registry.access.redhat.com/ubi8/python-39:latest sleep 3600
podman ps   # testapp is running

# Try to remove image (FAILS)
podman rmi registry.access.redhat.com/ubi8/python-39:latest
# Error: Image used by container abc123

# Graceful removal: stop container first
podman stop testapp
podman rm testapp
podman rmi registry.access.redhat.com/ubi8/python-39:latest   # now works

# OR force removal: automatically stops and removes container
podman run -d --name testapp2 registry.access.redhat.com/ubi8/python-39:latest sleep 3600
podman rmi -f registry.access.redhat.com/ubi8/python-39:latest
# Container testapp2 is stopped and removed automatically!
podman ps -a | grep testapp2   # gone!
```

---

## Security Perspective

### Image Security — What the Book Does Not Tell You

**1. The supply chain risk:**
Every image you use came from somewhere. An unverified image from Docker Hub could contain:
- Cryptocurrency miners
- Backdoors and reverse shells
- Data exfiltration code
- Hardcoded credentials

**2. Base image vulnerabilities:**
Your application is only as secure as its base image. Check the Security tab on catalog.redhat.com before using any image.

```bash
# Check image vulnerability grade (via Red Hat Catalog)
# A = excellent, F = critical vulnerabilities present
# Never deploy images graded D or F without security team approval

# Use Skopeo to inspect before pulling
skopeo inspect docker://registry.access.redhat.com/ubi8/python-39:latest | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d.get('Labels',{}), indent=2))"
# Look for: release, version, architecture labels
```

**3. The auth.json risk:**
```bash
# auth.json stores credentials in BASE64 — NOT encrypted
# Risk: file is readable by any process running as your user

# Mitigation 1: Use short-lived tokens instead of passwords
podman login --username myuser --password-stdin registry.redhat.io < token.txt

# Mitigation 2: Use credential helpers
# Configure in ~/.config/containers/auth.json to use keychain

# Mitigation 3: Logout after operations
podman logout registry.redhat.io

# Mitigation 4: Use service accounts (in CI/CD)
# Never store human credentials in CI/CD pipelines
# Use bot accounts with minimal permissions
```

**4. Registry blocking:**
```bash
# Block public registries in enterprise environments
cat > ~/.config/containers/registries.conf << 'EOF'
unqualified-search-registries = ['registry.company.internal']

[[registry]]
location = "docker.io"
blocked = true   # no pulls from Docker Hub

[[registry]]
location = "registry.company.internal"
insecure = false  # require TLS
EOF
```

**5. UBI vs arbitrary images:**
```bash
# PREFER: Red Hat UBI images — CVE-tracked, enterprise-supported
registry.access.redhat.com/ubi8:latest

# AVOID in production: unverified Docker Hub images
docker.io/someperson/someimage:latest  # unknown provenance

# VERIFY: Check image labels for provenance
podman inspect ubi8:latest --format '{{range $k,$v := .Config.Labels}}{{$k}}={{$v}}
{{end}}'
# Look for: vendor=Red Hat, name, version, release, build-date
```

---

## Enterprise Best Practices

### What Senior Engineers Know (from 30 years of experience)

```
1. ALWAYS use qualified image names in production
   ❌ nginx:latest
   ✅ registry.access.redhat.com/ubi8/nginx-118:1-111

2. NEVER use :latest in production Kubernetes/OpenShift manifests
   ❌ image: myapp:latest
   ✅ image: myapp:v2.5.3
   ✅ image: myapp@sha256:abc123...

3. ALWAYS scan images before deployment
   → Use Red Hat Catalog security tab
   → Use Quay.io's built-in scanning
   → Enterprise: Aqua Security, Twistlock, Sysdig

4. Maintain your own internal mirror registry
   → Pull from Red Hat once, push to internal registry
   → Reduces internet dependency
   → Enables access control
   → Examples: Quay Enterprise, Harbor, Nexus, Artifactory

5. Use image tags as part of your deployment strategy
   → Dev: myapp:dev-2024-01-15
   → Staging: myapp:rc-2.5.0
   → Production: myapp:2.5.0 or @sha256:...

6. Implement image lifecycle policies
   → Delete images older than 90 days from dev registries
   → Keep production images for 2+ years (audit/rollback)
   → automate with Quay's lifecycle policies

7. Never store secrets in images
   → Use OpenShift Secrets, Vault, or AWS Secrets Manager
   → Image layers are readable by anyone with image access

8. Pull once, use everywhere
   → Build in CI/CD once
   → Promote the SAME image SHA through environments
   → Dev → Test → Staging → Prod = same image bytes
```

---

## Troubleshooting Guide

### Troubleshooting Scenario 1: "Image not found" error

```
Symptom: Error: trying to pull docker.io/nginx — image not known

Investigation:
1. Check if image name is correct:
   podman search nginx
   skopeo list-tags docker://registry.access.redhat.com/ubi8

2. Check if registry is reachable:
   curl -I https://registry.access.redhat.com/v2/

3. Check registries.conf for typos:
   cat /etc/containers/registries.conf

4. Try with fully qualified name:
   podman pull registry.access.redhat.com/ubi8/httpd-24:latest
```

### Troubleshooting Scenario 2: Authentication failure

```
Symptom: Error: unauthorized: access denied

Investigation:
1. Check if you are logged in:
   podman login --get-login registry.redhat.io

2. Check credentials file:
   cat ${XDG_RUNTIME_DIR}/containers/auth.json

3. Try logging in again:
   podman login registry.redhat.io

4. Verify the image exists for your subscription:
   # UBI images are free — no subscription needed
   # RHEL product images need a subscription
```

### Troubleshooting Scenario 3: Cannot remove image

```
Symptom: Error: Image used by container XYZ

Investigation:
1. Find which container uses the image:
   podman ps -a --filter ancestor=IMAGE_NAME

2. Stop and remove the container:
   podman stop CONTAINER_NAME
   podman rm CONTAINER_NAME

3. Then remove the image:
   podman rmi IMAGE_NAME

4. OR force remove (kills container automatically):
   podman rmi -f IMAGE_NAME
```

### Troubleshooting Scenario 4: Image pull is very slow

```
Symptom: Image pull takes 30+ minutes

Investigation:
1. Check network:
   curl -o /dev/null -s -w "%{speed_download}\n" https://registry.access.redhat.com/

2. Check if same image is in local storage (save bandwidth):
   podman images | grep IMAGE_NAME

3. Check if layers are being reused:
   # Pull output shows "done" quickly for cached layers
   # "Copying blob" with progress = downloading new layer

4. Use internal registry to cache images:
   podman pull internal-registry.company.com/ubi8:latest
   # Much faster on corporate network!
```

### Troubleshooting Scenario 5: Wrong image running in production

```
Symptom: Production behaves differently from what was deployed

Investigation:
1. Check what image is actually running:
   podman inspect CONTAINER_NAME --format "{{.ImageName}}"
   podman inspect CONTAINER_NAME --format "{{.Image}}"  # digest

2. Compare digest with expected:
   podman inspect CONTAINER_NAME --format "{{.ImageDigest}}"
   # vs what your CI/CD pipeline built

3. Check image history:
   podman history IMAGE_ID

4. Next time: use digests in deployment specs
   podman run registry.company.com/myapp@sha256:abc123... 
   # sha256 digest cannot be changed or substituted
```

---

## Interview Preparation

### Beginner Questions

**Q: What is a container registry?**
A: A server that stores, manages, and distributes container images. Like GitHub for code, but for container images.

**Q: What is the difference between an image and a container?**
A: An image is a read-only blueprint. A container is a running instance of an image — like a class (image) vs an object (container) in OOP.

**Q: How do you pull an image with Podman?**
A: `podman pull registry.access.redhat.com/ubi8:latest`

**Q: What is UBI?**
A: Universal Base Image — a free, redistributable subset of RHEL that can be used as a base for container applications. No subscription required to use or distribute.

**Q: What does :latest mean in an image tag?**
A: It's the default tag when none is specified. It's considered bad practice because it changes whenever the maintainer releases a new version.

---

### Intermediate Questions

**Q: What is the difference between registry.access.redhat.com and registry.redhat.io?**
A: access.redhat.com is free, no auth needed, contains UBI images. registry.redhat.io requires a Red Hat account and contains full RHEL product images.

**Q: How does Podman resolve unqualified image names?**
A: It reads `/etc/containers/registries.conf` and searches the `unqualified-search-registries` list in order.

**Q: What is a dangling image?**
A: An image with no tag and not referenced by any other image. Created when rebuilding an image using an existing tag. Removed with `podman image prune`.

**Q: How do you inspect a remote image without downloading it?**
A: `skopeo inspect docker://registry.access.redhat.com/ubi8:latest`

**Q: Where does Podman store registry credentials?**
A: In `${XDG_RUNTIME_DIR}/containers/auth.json`, base64-encoded.

---

### Advanced / Scenario Questions

**Q: Your CI/CD pipeline uses :latest tags and production keeps breaking after image updates. How do you fix this?**
A: Switch to semantic version tags or image digests. Update pipeline to:
1. Build and tag with version: `podman build -t myapp:v$(cat VERSION) .`
2. Also tag as commit SHA: `podman tag myapp:v1.2.3 myapp:$(git rev-parse --short HEAD)`
3. Push both tags
4. Update deployment manifests to use the version tag, not :latest
5. For production: use `@sha256:` digest for guaranteed immutability

**Q: How do you set up Podman to use only your company's internal registry?**
A: Configure `/etc/containers/registries.conf`:
```
unqualified-search-registries = ['registry.company.internal']
[[registry]]
location = "docker.io"
blocked = true
```

**Q: A developer deployed the wrong version of an image to production. How do you roll back?**
A: 
1. `podman inspect CONTAINER --format "{{.Image}}"` — get running image digest
2. Pull the previous known-good version by its digest or version tag
3. Stop current container: `podman stop CONTAINER`
4. Start new container with previous image
5. Verify functionality
6. In future: maintain image promotion log with digests

---

## Certification Exam Tips

### EX188 Exam Key Points for Chapter 3

```
1. Know both ways to list images:
   podman images   AND   podman image ls

2. Know both ways to remove images:
   podman rmi IMAGE   AND   podman image rm IMAGE

3. Go template syntax is frequently tested:
   podman inspect IMAGE --format "{{.Config.Cmd}}"
   podman inspect IMAGE --format "{{.Config.User}}"
   podman inspect IMAGE --format "{{.NetworkSettings.Ports}}"

4. Know the difference:
   podman image prune      → dangling only
   podman image prune -a   → dangling + unused
   podman rmi --all        → removes ALL images

5. Registry authentication flow:
   podman login REGISTRY
   podman pull REGISTRY/IMAGE
   podman push REGISTRY/IMAGE

6. Where credentials are stored:
   ${XDG_RUNTIME_DIR}/containers/auth.json

7. Unqualified name resolution:
   /etc/containers/registries.conf
   unqualified-search-registries = [...]

8. Tag syntax:
   podman tag SOURCE:TAG DEST:TAG
   Multiple tags = same image ID

9. Build + tag + push workflow:
   podman build -f Containerfile -t quay.io/user/app:v1 .
   podman push quay.io/user/app:v1

10. Image storage location:
    User: ~/.local/share/containers
    Root: /var/lib/containers
```

### Common Exam Traps

```
TRAP 1: podman image prune vs podman image prune -a
  → Without -a: only dangling (untagged) images
  → With -a: all images not used by any container

TRAP 2: -f on rmi vs prune
  → podman rmi -f: force removes even if container uses it
  → podman image prune -f: skip interactive prompt (no confirmation)

TRAP 3: podman tag does not MOVE a tag — it CREATES a new one
  → Source tag still exists after podman tag

TRAP 4: Root images not visible to regular user
  → Images pulled by root: /var/lib/containers
  → Not shown in podman images as regular user

TRAP 5: :latest is NOT always the newest
  → :latest is just a tag name — maintainers can tag ANY image as :latest
  → Never assume :latest = most recent
```

---

## Key Concepts Summary

### The 7 Core Concepts of Chapter 3

```
1. REGISTRIES store images
   Red Hat provides two: access.redhat.com (free) and redhat.io (auth required)
   Use catalog.redhat.com to browse, search, and check security

2. UBI = Enterprise-grade free base images
   Based on RHEL, OCI compliant, no subscription to use or distribute
   Full Red Hat support only when deployed on RHOCP/RHEL

3. TAGS = version pointers
   Multiple tags can point to the same image ID
   :latest is a bad practice — use semantic versions or digests

4. AUTH = base64-stored credentials
   podman login saves to ${XDG_RUNTIME_DIR}/containers/auth.json
   Base64 encoded (NOT encrypted) — protect the file!

5. SEARCH = registries.conf
   Unqualified names are searched in order from registries.conf
   Always use fully qualified names for predictability

6. INSPECT = Go templates
   podman inspect --format "{{.Config.Cmd}}" = extract specific metadata
   skopeo inspect = inspect WITHOUT pulling

7. PRUNE = storage management
   prune alone = dangling only
   prune -a = all unused
   prune -f = no confirmation prompt
```

---

## Chapter Summary & Cheat Sheet

### One-Page Revision

```bash
# ── REGISTRIES ───────────────────────────────────────────────────────────
registry.access.redhat.com   # Free, no login, UBI images
registry.redhat.io           # Login required, full RHEL images
quay.io                      # Store your own images (free for public)
catalog.redhat.com           # Search, security scan, Containerfile viewer

# ── AUTH ─────────────────────────────────────────────────────────────────
podman login REGISTRY        # login (password hidden as you type)
podman logout REGISTRY       # logout
cat ${XDG_RUNTIME_DIR}/containers/auth.json  # view stored credentials

# ── SEARCH ───────────────────────────────────────────────────────────────
podman search TERM           # search all registries in registries.conf
skopeo list-tags docker://REGISTRY/IMAGE  # list available tags (no pull)
skopeo inspect docker://IMAGE             # inspect without pulling

# ── PULL ─────────────────────────────────────────────────────────────────
podman pull IMAGE:TAG        # download image
podman pull IMAGE@sha256:... # pull by digest (immutable!)
podman images                # list local images
podman image ls              # same

# ── BUILD ────────────────────────────────────────────────────────────────
podman build -f Containerfile -t REGISTRY/USER/IMAGE:TAG .
podman push REGISTRY/USER/IMAGE:TAG

# ── TAG ──────────────────────────────────────────────────────────────────
podman tag SOURCE:TAG DEST:NEW_TAG    # create new tag (source stays)
podman images | grep IMAGE            # verify all tags

# ── INSPECT ──────────────────────────────────────────────────────────────
podman inspect IMAGE --format "{{.Config.Cmd}}"
podman inspect IMAGE --format "{{.Config.User}}"
podman inspect IMAGE --format "{{.Config.Env}}"
podman inspect IMAGE --format "{{.Config.ExposedPorts}}"
podman inspect IMAGE --format "{{.Digest}}"
podman history IMAGE         # view layer history

# ── REMOVE ───────────────────────────────────────────────────────────────
podman rmi IMAGE             # remove image (fails if container uses it)
podman rmi -f IMAGE          # force: stops containers, removes, removes image
podman rmi --all             # remove ALL local images
podman image prune           # dangling images only
podman image prune -a        # dangling + all unused
podman image prune -af       # same but no confirmation prompt
podman system df             # disk usage
```

---

## My Learning Progress

| Topic | Status | Commands Practiced |
|-------|--------|--------------------|
| Container registries | ✅ Complete | login, logout, search |
| Red Hat Catalog | ✅ Complete | Web UI exploration |
| UBI images | ✅ Complete | pull, inspect, compare variants |
| Quay.io | ✅ Complete | login, build via UI, push |
| registries.conf | ✅ Complete | Configuration, unqualified names |
| Registry auth + auth.json | ✅ Complete | login, credential inspection |
| Image versioning + tags | ✅ Complete | tag, semantic versioning |
| Pull images | ✅ Complete | pull, image ls, storage locations |
| Build images | ✅ Complete | build, Containerfile |
| Push images | ✅ Complete | push, login first |
| Inspect images | ✅ Complete | inspect, --format, Go templates |
| Remove images | ✅ Complete | rmi, prune, -a, -f flags |
| Dangling images | ✅ Complete | prune, filter dangling=true |
| skopeo | ✅ Complete | inspect, list-tags |
| Basic labs | ✅ Complete | 7 labs |
| Scenario labs | ✅ Complete | 5 labs |
| Intermediate labs | ✅ Complete | 5 labs |
| Interview Q&A | ✅ Complete | 10 Q&A with solutions |

**Guided Exercises from book completed:**
- [x] images-basics (Quay + Red Hat Catalog exploration)
- [x] images-managing (pull, search, run, prune)
- [x] images-lab (build on Quay, pull, tag, run)

---

## Resources

- [Red Hat Container Catalog](https://catalog.redhat.com/software/containers/explore)
- [Quay.io](https://quay.io)
- [Red Hat DO188 Course](https://www.redhat.com/en/services/training/do188)
- [Skopeo Documentation](https://github.com/containers/skopeo)
- [Manage Container Registries — Red Hat](https://www.redhat.com/sysadmin/manage-container-registries)
- [Podman Image Commands](https://docs.podman.io/en/latest/markdown/podman-image.1.html)
- [EX188 Exam Objectives](https://www.redhat.com/en/services/training/ex188-red-hat-certified-specialist-containers-exam)
- [registries.conf Documentation](https://www.man7.org/linux/man-pages/man5/containers-registries.conf.5.html)

---

*Study journal maintained by **Ahmad Ali** | MNS University of Agriculture, Multan, Pakistan*  
*DO188 Red Hat OpenShift Development I — Chapter 3: Container Images*  
*Mentor: Ali Ijaz*
