# Chapter 3 - Container Images

**Course:** DO188 - Red Hat OpenShift Developer I: Introduction to Containers with Podman  
**Chapter:** 3 - Container Images    
**Goal:** Navigate container registries to find and manage container images.

## How to Use This README

Use this file as your Chapter 3 GitHub portfolio note, daily practice guide, interview preparation sheet, and certification revision file. The content is based on Chapter 3 of the DO188 workbook, then expanded with enterprise, security, DevOps, OpenShift, Kubernetes, and SRE context.

## Table of Contents

- [Section 1: Chapter Analysis](#section-1-chapter-analysis)
- [Section 2: Concept Mastery](#section-2-concept-mastery)
- [Section 3: Enterprise Experience](#section-3-enterprise-experience)
- [Section 4: Security Perspective](#section-4-security-perspective)
- [Section 5: Command Deep Dive](#section-5-command-deep-dive)
- [Section 6: Lab-Driven Learning](#section-6-lab-driven-learning)
- [Section 7: Interview Preparation](#section-7-interview-preparation)
- [Section 8: Certification Preparation](#section-8-certification-preparation)
- [Section 9: Career Perspective](#section-9-career-perspective)
- [Section 10: Beyond the Chapter](#section-10-beyond-the-chapter)
- [Section 11: Final Mastery Guide](#section-11-final-mastery-guide)

# Section 1: Chapter Analysis

## Complete Chapter Outline

Chapter 3 focuses on finding, storing, pulling, tagging, inspecting, building, pushing, and removing container images.

## Major Topics

| Major Topic | What It Covers |
| --- | --- |
| Container Image Registries | Registry purpose, Red Hat Registry, Red Hat Catalog, Quay.io, UBI images, registry configuration, registry authentication |
| Managing Images | Image tags, semantic versioning, pulling images, listing images, building images, pushing images, inspecting images, removing images, pruning images |

## Subtopics

### Container Image Registries

- What a container image registry is
- Examples of registries
- Red Hat Registry
- Red Hat Catalog
- Quay.io
- Universal Base Images
- Containerfile basics
- Registry search and technical metadata
- `registries.conf`
- Unqualified image names
- Blocking registries
- Registry authentication
- Podman credential storage

### Managing Images

- Image lifecycle
- Image repositories, namespaces, image names, and tags
- Semantic versioning
- Why `latest` is risky
- Image search
- Image pull
- Image list
- Image build
- Image tag
- Image push
- Image inspect
- Image remove
- Image prune

## Commands Found in Chapter 3

| Command | Purpose |
| --- | --- |
| `podman pull IMAGE` | Pull an image from a registry |
| `podman image pull IMAGE` | Pull an image by using the image subcommand |
| `podman images` | List local images |
| `podman image ls` | List local images |
| `podman search TERM` | Search configured registries |
| `podman login REGISTRY` | Authenticate to a registry |
| `podman build -f Containerfile -t IMAGE` | Build an image from a Containerfile |
| `podman image tag SOURCE TARGET` | Add another tag to an image |
| `podman tag SOURCE TARGET` | Short form for tagging an image |
| `podman push IMAGE` | Push an image to a remote registry |
| `podman image inspect IMAGE` | Inspect image metadata |
| `podman image rm IMAGE` | Remove a local image |
| `podman rmi IMAGE` | Short form for image removal |
| `podman image prune` | Remove dangling images |
| `podman image prune -a` | Remove unused images |
| `podman image prune -af` | Remove unused images without prompt |
| `podman run IMAGE COMMAND` | Test an image by creating a container |
| `podman run -d -p HOST:CONTAINER IMAGE` | Run image as a network service |

## Chapter Workflows

### Registry Discovery Workflow

```text
Find image -> Check source registry -> Check tags -> Review metadata -> Pull image -> Test image
```

### Image Management Workflow

```text
Search -> Pull -> List -> Inspect -> Tag -> Run -> Remove or Push
```

### Build and Share Workflow

```text
Create Containerfile -> Build image -> Tag image -> Login to registry -> Push image -> Pull image elsewhere -> Run container
```

### Cleanup Workflow

```text
List containers -> Stop/remove dependent containers -> Remove image tags -> Remove image -> Prune unused images
```

# Section 2: Concept Mastery

## Topic 1: Container Image Registries

### Definition

A container image registry is a storage and distribution service for container images.

### What Is It?

It is like a software warehouse. Developers and platform teams push images into the registry, and servers, CI/CD pipelines, OpenShift, Kubernetes, and Podman pull images from it.

### Purpose

The registry exists so teams can share images in a controlled and repeatable way.

### Problem It Solves

Without a registry, every server would need to build or copy images manually. That does not scale in real environments.

### Importance

Registries are central to container delivery. If the registry is unavailable or insecure, deployments can fail or unsafe images can enter production.

### Red Hat Perspective

Red Hat teaches registries because OpenShift, Podman, and enterprise container platforms depend on trusted image sources. Red Hat also provides official and certified images through Red Hat registries and catalog services.

### Beginner Explanation

Think of a registry like GitHub for container images. You do not manually move application packages everywhere. You upload the image once, then pull it when needed.

### Intermediate Explanation

An image reference usually contains:

```text
REGISTRY/NAMESPACE/IMAGE_NAME:TAG
```

Example:

```text
registry.access.redhat.com/ubi8/python-39:latest
```

The registry is the server. The namespace or organization groups images. The image name identifies the artifact. The tag identifies the version.

### Industry Explanation

In enterprise environments, registries are controlled infrastructure. Companies often use Red Hat Quay, OpenShift internal registry, Harbor, Artifactory, Nexus, Amazon ECR, Azure Container Registry, or Google Artifact Registry. Access is controlled by IAM, robot accounts, service accounts, and CI/CD credentials.

## Topic 2: Red Hat Registry and Red Hat Catalog

### Definition

Red Hat provides container images through official Red Hat registries and makes them searchable through the Red Hat Catalog.

### Purpose

The purpose is to provide trusted enterprise images, technical metadata, package information, tags, and security details.

### Problem It Solves

Public registries can contain untrusted, outdated, or vulnerable images. Red Hat images give enterprises a trusted base.

### Importance

For Red Hat exams and production environments, trusted images matter. OpenShift administrators and DevSecOps teams usually prefer vendor-supported or internally approved images.

### Red Hat Perspective

Red Hat teaches this because DO188 students must learn to identify official images, inspect their metadata, and understand why UBI images are important.

### Beginner Explanation

The Red Hat Catalog is where you search for Red Hat container images and read information about them.

### Intermediate Explanation

The Catalog can show:

- Image overview
- Security health
- Technical details
- Architecture
- Exposed ports
- Installed packages
- Dockerfile or Containerfile data
- Pull commands for Podman, Docker, and OpenShift

### Industry Explanation

Large companies often maintain an approved image list. Security teams review image sources and allow only approved registries. Red Hat Catalog data helps with that approval process.

## Topic 3: Universal Base Images

### Definition

Universal Base Images are Red Hat base images that contain a subset of RHEL content and can be used as base layers for applications.

### Purpose

UBI gives developers a RHEL-compatible base image for building applications.

### Problem It Solves

Developers need a reliable base image. Random community images can introduce unknown packages, missing support, or security risk.

### Importance

UBI images are common in Red Hat and OpenShift environments.

### Red Hat Perspective

Red Hat teaches UBI because it is the recommended base family for many Red Hat-based container workflows.

### Beginner Explanation

UBI is a small Red Hat Linux foundation for your container image.

### Intermediate Explanation

UBI images include runtime variants such as Python, Node.js, and Go toolset images. They can be pulled from Red Hat registries and used as parent images in Containerfiles.

### Industry Explanation

Enterprises use standard base images so patching, scanning, support, and compliance are easier. A platform team may publish a company-approved UBI image with extra certificates, security settings, and monitoring tools.

## Topic 4: Quay.io

### Definition

Quay.io is a container image registry where users and organizations can store and distribute images.

### Purpose

It gives developers a place to host their own container images.

### Problem It Solves

Teams need a registry for images they build themselves, not only vendor-provided images.

### Importance

Quay is common in Red Hat ecosystems and supports enterprise image workflows.

### Red Hat Perspective

Red Hat teaches Quay because it is part of the Red Hat container platform ecosystem and is useful for sharing images.

### Beginner Explanation

Quay.io is a website where you can create repositories for container images.

### Intermediate Explanation

You can create public or private image repositories, build from a Containerfile, push images with Podman, and pull them into local or cluster environments.

### Industry Explanation

Enterprises may use Red Hat Quay on-premises for disconnected environments, regulated workloads, internal image scanning, image promotion, and audit control.

## Topic 5: Registry Configuration

### Definition

Registry configuration tells Podman where to search for images when a full registry name is not provided.

### Purpose

It makes image pulling easier and lets administrators control allowed or blocked registries.

### Problem It Solves

Unqualified image names can be ambiguous. For example, `python` could exist in many registries.

### Importance

Registry configuration helps avoid pulling from the wrong source.

### Red Hat Perspective

Red Hat wants administrators and developers to understand how Podman decides which registries to use.

### Beginner Explanation

If you do not tell Podman the full address, Podman checks its registry search list.

### Intermediate Explanation

The `/etc/containers/registries.conf` file can define search registries and blocked registries.

Example idea:

```toml
unqualified-search-registries = ['registry.redhat.io', 'docker.io']
```

### Industry Explanation

Enterprises often block public registries directly from production servers. They force developers and clusters to pull from trusted internal registries only.

## Topic 6: Registry Authentication

### Definition

Registry authentication is the process of logging in to a registry before pulling or pushing restricted images.

### Purpose

It protects private images and restricted vendor content.

### Problem It Solves

Without authentication, anyone could pull or push sensitive images.

### Importance

Authentication is required for private registries, protected images, and push operations.

### Red Hat Perspective

Red Hat teaches `podman login` because some Red Hat images and Quay workflows require authentication.

### Beginner Explanation

Before downloading or uploading private images, you log in.

### Intermediate Explanation

Podman stores registry login information in a user-specific authentication file. The password is not shown in the terminal when typed.

### Industry Explanation

Production environments use robot accounts, service accounts, CI/CD secrets, Kubernetes image pull secrets, short-lived tokens, and registry access policies.

## Topic 7: Image Tags and Versioning

### Definition

An image tag identifies a version or variant of an image.

### Purpose

Tags help teams deploy the correct application version.

### Problem It Solves

Without tags, teams cannot easily distinguish image versions.

### Importance

Tags directly affect deployment reproducibility.

### Red Hat Perspective

Red Hat teaches tagging because exam tasks and real workflows require creating and using image tags.

### Beginner Explanation

A tag is like a label. `myapp:1.0` and `myapp:2.0` can point to different versions.

### Intermediate Explanation

One image ID can have multiple tags. Removing one tag does not always remove the image layers if another tag still references them.

### Industry Explanation

Production teams avoid relying on `latest`. They use version tags, Git SHA tags, release tags, and sometimes immutable digest references.

## Topic 8: Image Management

### Definition

Image management is the set of operations used to search, pull, list, build, tag, push, inspect, remove, and prune images.

### Purpose

It controls the full image lifecycle on a developer machine or server.

### Problem It Solves

Without image management, local storage becomes messy, deployments become inconsistent, and troubleshooting becomes harder.

### Importance

Image management is a core Podman and DO188 skill.

### Red Hat Perspective

Red Hat teaches this because container images are deployment artifacts. Managing them correctly is required for container operations.

### Beginner Explanation

You download images, check them, give them names, run them, and remove them when no longer needed.

### Intermediate Explanation

Podman stores user images in user-specific container storage. Root and non-root users have separate image stores.

### Industry Explanation

Enterprises treat images like release artifacts. They are built by CI, scanned, signed, promoted through environments, and eventually retired.

# Section 3: Enterprise Experience

## What Beginners Usually Misunderstand

- They think `latest` means stable. It does not.
- They think image names are always unique. They are only unique when fully qualified.
- They think deleting a tag always deletes the whole image. It might only remove one reference.
- They think public images are automatically safe.
- They confuse pulling an image with running a container.
- They forget that root and rootless Podman use different local image stores.

## What the Course Does Not Explain Deeply

- Image digests and immutable references
- Image signing and verification
- CI/CD image promotion workflows
- Registry replication
- Vulnerability management at scale
- Image retention policies
- Air-gapped registry workflows
- Kubernetes `ImagePullBackOff` troubleshooting

## What Senior Engineers Know

- The registry is part of the production platform, not just a storage location.
- Tagging strategy must be designed before production.
- `latest` breaks repeatability.
- Internal base images reduce risk and standardize operations.
- Image scanning must be integrated into pipelines.
- Image cleanup policies prevent storage exhaustion.
- Authentication and authorization must use least privilege.

## Production Examples

| Industry | Real Usage |
| --- | --- |
| Bank | Uses internal Quay or Artifactory. Only approved and scanned images can reach production. |
| Telecom | Mirrors vendor images into an internal registry for reliability and disconnected sites. |
| Cloud Provider | Uses automated pipelines to build, scan, sign, replicate, and deploy images globally. |
| SaaS Company | Tags every image with Git SHA, semantic version, and environment promotion label. |
| Enterprise DevOps Team | Maintains golden UBI base images and blocks direct Docker Hub pulls. |

## Common Enterprise Mistakes

- Using unqualified image names in production manifests
- Pulling directly from public registries in production
- Sharing personal credentials in CI/CD
- Not cleaning old images
- Not scanning images
- Not pinning versions
- Rebuilding images without traceability
- Using the same tag for different builds

## Best Practices

- Use fully qualified image references.
- Prefer trusted registries.
- Avoid `latest` for production.
- Use semantic versioning or Git SHA tags.
- Scan images before deployment.
- Keep base images patched.
- Use service accounts or robot accounts for automation.
- Use private registries for internal images.
- Document image ownership.
- Apply retention policies.

# Section 4: Security Perspective

## Registry Security Risks

- Pulling malicious public images
- Pushing unauthorized images
- Leaked registry credentials
- Weak registry access controls
- Using outdated base images
- Using images with vulnerable packages
- Using mutable tags without audit
- Man-in-the-middle risk if TLS is misconfigured

## Common Vulnerabilities

- Vulnerable OS packages inside images
- Exposed secrets accidentally copied into images
- Old runtime versions
- Unpatched base images
- Overly large images with unnecessary tools
- Images running as root
- Unknown or untrusted image publishers

## Attack Vectors

- Typosquatting image names
- Poisoned public images
- Stolen registry tokens
- CI/CD pipeline compromise
- Pushing a malicious image under a trusted tag
- Pulling from an unapproved registry

## Security Best Practices

- Use Red Hat UBI or approved base images.
- Pull from trusted registries only.
- Require authentication for private registries.
- Use least privilege credentials.
- Scan images for vulnerabilities.
- Avoid embedding secrets in images.
- Pin image versions.
- Use image signing and verification where available.
- Use registry audit logs.
- Block untrusted registries in policy.

## Image Hardening

- Use minimal base images.
- Remove unnecessary tools and packages.
- Keep dependencies updated.
- Run as non-root where possible.
- Use trusted parent images.
- Avoid shell scripts that download random internet content during builds.

## Supply Chain Security

Image supply chain security means protecting the path from source code to built image to registry to deployment.

Important controls:

- Source control review
- Controlled CI/CD runners
- Build logs
- SBOM generation
- Vulnerability scanning
- Image signing
- Registry access control
- Deployment admission policies

# Section 5: Command Deep Dive

## `podman login REGISTRY`

**Purpose:** Authenticate to a registry.

**Syntax:**

```bash
podman login REGISTRY
```

**Example:**

```bash
podman login quay.io
podman login registry.redhat.io
```

**Expected Output:**

```text
Login Succeeded!
```

**Common Errors:**

- Wrong username or password
- Registry unreachable
- TLS certificate issue
- Account lacks permissions

**Troubleshooting:**

```bash
podman logout quay.io
podman login quay.io
```

**Exam Tip:** Know when authentication is required. Pulling public UBI images might work without login, but private images and push operations usually require login.

## `podman pull IMAGE` and `podman image pull IMAGE`

**Purpose:** Download an image from a registry.

**Syntax:**

```bash
podman pull REGISTRY/NAMESPACE/IMAGE:TAG
podman image pull REGISTRY/NAMESPACE/IMAGE:TAG
```

**Example:**

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest
```

**Expected Output:** Pull progress, copied layers, manifest write, and signature storage.

**Common Errors:**

- Image not found
- Wrong tag
- Authentication failure
- Registry unreachable

**Best Practice:** Use a full image reference and a specific tag.

## `podman images` and `podman image ls`

**Purpose:** List local images.

**Syntax:**

```bash
podman images
podman image ls
```

**Useful Format Example:**

```bash
podman image ls --format "{{.Repository}}:{{.Tag}} {{.ID}}"
```

**Exam Tip:** Use formatted output when you need exact repository names.

## `podman search TERM`

**Purpose:** Search configured registries.

**Syntax:**

```bash
podman search python
podman search nginx
```

**Important:** Search depends on registries configured in `registries.conf`.

**Best Practice:** Search is useful, but for production use, verify image details in an approved registry catalog.

## `podman build -f Containerfile -t IMAGE`

**Purpose:** Build an image from a Containerfile.

**Syntax:**

```bash
podman build -f Containerfile -t IMAGE_NAME:TAG
```

**Example:**

```bash
podman build -f Containerfile -t simple-server:0.1
```

**Expected Output:** Build steps, intermediate layers, commit, and tag.

**Common Errors:**

- Containerfile path wrong
- Base image unavailable
- Build command fails
- Network problem during build

**Exam Tip:** If no tag is specified, `latest` is used.

## `podman image tag SOURCE TARGET`

**Purpose:** Add another name or tag to an existing image.

**Syntax:**

```bash
podman image tag SOURCE_IMAGE TARGET_IMAGE
```

**Example:**

```bash
podman image tag simple-server simple-server:0.1
```

**Expected Output:** Usually no output.

**Important:** Multiple tags can point to the same image ID.

## `podman push IMAGE`

**Purpose:** Upload an image to a registry.

**Syntax:**

```bash
podman push REGISTRY/NAMESPACE/IMAGE:TAG
```

**Example:**

```bash
podman push quay.io/YOUR_USER/simple-server:0.1
```

**Common Errors:**

- Not logged in
- Repository does not exist
- Permission denied
- Wrong image name

**Best Practice:** Tag the image with the remote registry path before pushing.

## `podman image inspect IMAGE`

**Purpose:** View image metadata.

**Syntax:**

```bash
podman image inspect IMAGE
```

**Format Example:**

```bash
podman image inspect IMAGE --format="{{.Config.Cmd}}"
```

**Useful Metadata:**

- Default command
- Entrypoint
- Environment variables
- Exposed ports
- Working directory
- User
- Labels
- Architecture

## `podman image rm IMAGE` and `podman rmi IMAGE`

**Purpose:** Remove local images.

**Syntax:**

```bash
podman image rm IMAGE
podman rmi IMAGE
```

**Force Removal:**

```bash
podman image rm -f IMAGE
```

**Common Error:** Image is used by a container.

**Resolution:**

```bash
podman ps --all
podman stop CONTAINER
podman rm CONTAINER
podman image rm IMAGE
```

## `podman image prune`

**Purpose:** Clean unused or dangling images.

**Examples:**

```bash
podman image prune
podman image prune -a
podman image prune -af
```

**Warning:** `-a` removes all images without associated containers.

# Section 6: Lab-Driven Learning

## Beginner Labs - Container Image Registries

### Beginner Lab 1 - Pull a UBI Image

**Objective:** Pull a trusted Red Hat UBI image.

**Prerequisites:** Podman installed and registry access.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/ubi:8.6
podman images
```

**Expected Output:** The UBI image appears locally.

**Validation:**

```bash
podman image ls | grep ubi
```

**Learning Outcome:** You can retrieve an image from a trusted registry.

### Beginner Lab 2 - Pull a Python Runtime Image

**Objective:** Pull a Red Hat Python image and test it.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman run --rm registry.access.redhat.com/ubi8/python-39:latest python --version
```

**Expected Output:** Python version is displayed.

**Learning Outcome:** You can pull and test a language runtime image.

### Beginner Lab 3 - Search for an Image

**Objective:** Search configured registries for Python images.

**Commands:**

```bash
podman search python
```

**Validation:** Identify at least one Red Hat or UBI-based Python image.

**Learning Outcome:** You understand registry search.

### Beginner Lab 4 - Explore Image Name Format

**Objective:** Identify registry, namespace, image, and tag.

**Example Image:**

```text
registry.access.redhat.com/ubi8/python-39:latest
```

**Answer:**

```text
Registry: registry.access.redhat.com
Namespace: ubi8
Image: python-39
Tag: latest
```

**Learning Outcome:** You can break down an image reference.

### Beginner Lab 5 - Login to a Registry

**Objective:** Authenticate with Quay.io.

**Commands:**

```bash
podman login quay.io
```

**Expected Output:**

```text
Login Succeeded!
```

**Learning Outcome:** You can authenticate before push or private pull.

## Beginner Labs - Managing Images

### Beginner Lab 6 - List Local Images

**Objective:** Display local images.

**Commands:**

```bash
podman image ls
podman images
```

**Learning Outcome:** You know both listing forms.

### Beginner Lab 7 - Inspect an Image

**Objective:** Inspect image metadata.

**Commands:**

```bash
podman image inspect registry.access.redhat.com/ubi8/python-39:latest
```

**Validation:** Find `Architecture`, `Env`, and `Cmd`.

### Beginner Lab 8 - Inspect Image Command Only

**Objective:** Use Go template output.

**Commands:**

```bash
podman image inspect registry.access.redhat.com/ubi8/python-39:latest --format="{{.Config.Cmd}}"
```

**Learning Outcome:** You can extract one metadata field.

### Beginner Lab 9 - Tag an Image

**Objective:** Add a semantic version tag.

**Commands:**

```bash
podman image tag registry.access.redhat.com/ubi8/python-39:latest python-runtime:0.1
podman image ls
```

**Validation:** `python-runtime:0.1` appears.

### Beginner Lab 10 - Remove a Tag

**Objective:** Remove a local tag.

**Commands:**

```bash
podman image rm python-runtime:0.1
podman image ls
```

**Learning Outcome:** You can remove a tag or image reference.

## Intermediate Labs

### Intermediate Lab 1 - Build a Simple Image

**Use Case:** Build a simple local web image from a Containerfile.

**Commands:**

```bash
mkdir images-managing-practice
cd images-managing-practice
printf 'FROM registry.access.redhat.com/ubi8/python-39:latest\nRUN echo "Hello from Chapter 3" > hello\nCMD python -m http.server\n' > Containerfile
podman build -f Containerfile -t simple-server:0.1
podman image ls
```

**Validation:**

```bash
podman image inspect simple-server:0.1 --format="{{.Config.Cmd}}"
```

**Troubleshooting:** If build fails, verify base image access and Containerfile syntax.

### Intermediate Lab 2 - Run Built Image

**Use Case:** Run the image as a web server.

**Commands:**

```bash
podman run -d --name simple-server -p 8080:8000 simple-server:0.1
curl http://localhost:8080/hello
```

**Validation:** The response shows your file content.

### Intermediate Lab 3 - Retag for a Registry

**Use Case:** Prepare an image for Quay.

**Commands:**

```bash
podman image tag simple-server:0.1 quay.io/YOUR_USER/simple-server:0.1
podman image ls
```

**Validation:** The image has a Quay.io tag.

### Intermediate Lab 4 - Push to Quay

**Use Case:** Share an image through a registry.

**Commands:**

```bash
podman login quay.io
podman push quay.io/YOUR_USER/simple-server:0.1
```

**Validation:** Confirm the image exists in Quay.

### Intermediate Lab 5 - Remove and Pull Back

**Use Case:** Prove the registry copy works.

**Commands:**

```bash
podman rm -f simple-server
podman image rm quay.io/YOUR_USER/simple-server:0.1
podman pull quay.io/YOUR_USER/simple-server:0.1
```

**Validation:** The image pulls successfully.

## Advanced Labs

### Advanced Lab 1 - Enforce Specific Tags

**Architecture:** Developer workstation pulls only explicitly versioned images.

**Workflow:** Pull, tag, run, and document a version.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman image tag registry.access.redhat.com/ubi8/python-39:latest approved-python:3.9
podman run --rm approved-python:3.9 python --version
```

**Validation:** The approved tag runs.

**Troubleshooting:** If the wrong tag runs, inspect image IDs.

### Advanced Lab 2 - Image Metadata Audit

**Architecture:** Security team asks for image metadata.

**Commands:**

```bash
podman image inspect approved-python:3.9 --format="{{.Architecture}}"
podman image inspect approved-python:3.9 --format="{{.Config.User}}"
podman image inspect approved-python:3.9 --format="{{.Config.Env}}"
podman image inspect approved-python:3.9 --format="{{.Config.Cmd}}"
```

**Validation:** You can report architecture, user, env, and command.

### Advanced Lab 3 - Cleanup Without Breaking Running Containers

**Architecture:** A server has old images and running containers.

**Commands:**

```bash
podman ps --all
podman image ls
podman image prune
```

**Validation:** Only dangling images are removed.

**Troubleshooting:** If images remain, they may be used by containers or tagged.

### Advanced Lab 4 - Force Cleanup Test

**Architecture:** A test image must be removed even if a container uses it.

**Commands:**

```bash
podman run -d --name cleanup-test simple-server:0.1
podman image rm -f simple-server:0.1
podman ps --all
podman image ls
```

**Validation:** The image and dependent container are removed.

### Advanced Lab 5 - Compare Tags Pointing to Same Image

**Architecture:** Release team creates multiple tags for the same artifact.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/ubi:8.6
podman image tag registry.access.redhat.com/ubi8/ubi:8.6 ubi-base:stable
podman image tag registry.access.redhat.com/ubi8/ubi:8.6 ubi-base:8.6
podman image ls
```

**Validation:** Tags share the same image ID.

## Professional Enterprise Labs

### Professional Lab 1 - CI/CD Image Build Workflow

**Architecture:** Source code repository -> CI runner -> Podman build -> Quay registry.

**Implementation:**

```bash
podman build -f Containerfile -t quay.io/YOUR_ORG/app:${GIT_COMMIT}
podman login quay.io
podman push quay.io/YOUR_ORG/app:${GIT_COMMIT}
```

**Validation:** Image exists in registry with Git commit tag.

### Professional Lab 2 - Image Promotion Workflow

**Architecture:** Same image promoted from dev to test to prod by retagging.

**Implementation:**

```bash
podman pull quay.io/YOUR_ORG/app:dev
podman image tag quay.io/YOUR_ORG/app:dev quay.io/YOUR_ORG/app:test
podman push quay.io/YOUR_ORG/app:test
```

**Validation:** The promoted tag points to the expected image ID.

### Professional Lab 3 - Approved Base Image Workflow

**Architecture:** Platform team publishes approved UBI-based base images.

**Implementation:**

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman image tag registry.access.redhat.com/ubi8/python-39:latest quay.io/YOUR_ORG/base-python:3.9
podman push quay.io/YOUR_ORG/base-python:3.9
```

**Validation:** Application teams can pull the approved base image.

### Professional Lab 4 - OpenShift Image Pull Planning

**Architecture:** OpenShift workload pulls from a private registry.

**Implementation Idea:**

```bash
podman login quay.io
podman pull quay.io/YOUR_ORG/app:1.0
```

**OpenShift Connection:** In a real cluster, create an image pull secret and attach it to the service account.

**Validation:** Deployment does not fail with image pull authentication errors.

### Professional Lab 5 - Registry Cleanup Policy

**Architecture:** Registry and local build systems need retention policies.

**Implementation:**

```bash
podman image ls
podman image prune -af
```

**Enterprise Note:** In production registries, cleanup is normally handled by retention policy, not manual deletion.

## Security Labs

### Security Lab 1 - Verify Image Source

**Risk:** Pulling from the wrong registry.

**Mitigation:** Use fully qualified image names.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/ubi:8.6
```

**Validation:** Confirm the repository starts with the expected registry.

### Security Lab 2 - Avoid `latest`

**Risk:** Unexpected updates.

**Mitigation:** Use a specific version tag.

**Commands:**

```bash
podman pull registry.access.redhat.com/ubi8/ubi:8.6
podman run --rm registry.access.redhat.com/ubi8/ubi:8.6 cat /etc/os-release
```

**Validation:** You know exactly which tag was used.

### Security Lab 3 - Check Image Metadata

**Risk:** Unknown default command, user, or exposed ports.

**Mitigation:** Inspect image before use.

**Commands:**

```bash
podman image inspect IMAGE --format="{{.Config.User}}"
podman image inspect IMAGE --format="{{.Config.ExposedPorts}}"
podman image inspect IMAGE --format="{{.Config.Cmd}}"
```

**Validation:** Document risky defaults.

### Security Lab 4 - Registry Authentication Test

**Risk:** Unauthorized access or broken credentials.

**Mitigation:** Test login.

**Commands:**

```bash
podman login quay.io
podman pull quay.io/YOUR_USER/YOUR_PRIVATE_IMAGE:TAG
```

**Validation:** Pull succeeds only with valid credentials.

### Security Lab 5 - Local Image Cleanup

**Risk:** Old vulnerable images remain on hosts.

**Mitigation:** Remove unused images.

**Commands:**

```bash
podman image prune -a
```

**Validation:** Unused images are removed after confirmation.

## Troubleshooting Labs

### Troubleshooting 1 - Image Not Found

**Symptoms:** Pull fails with image not found.

**Root Cause:** Wrong image name, namespace, registry, or tag.

**Investigation:**

```bash
podman pull IMAGE
podman search IMAGE_NAME
```

**Resolution:** Use the correct fully qualified image reference.

**Validation:** Pull succeeds.

### Troubleshooting 2 - Authentication Failure

**Symptoms:** Unauthorized error.

**Root Cause:** Not logged in or wrong credentials.

**Commands:**

```bash
podman login REGISTRY
podman pull REGISTRY/NAMESPACE/IMAGE:TAG
```

**Resolution:** Login with valid account or token.

### Troubleshooting 3 - Wrong Tag

**Symptoms:** Container runs an unexpected version.

**Root Cause:** Used `latest` or wrong tag.

**Commands:**

```bash
podman image ls
podman image inspect IMAGE --format="{{.Id}}"
```

**Resolution:** Pull and run the correct tag.

### Troubleshooting 4 - Registry Unreachable

**Symptoms:** Network timeout.

**Root Cause:** DNS, proxy, firewall, VPN, or registry outage.

**Commands:**

```bash
ping REGISTRY_HOST
curl -I https://REGISTRY_HOST
podman pull IMAGE
```

**Resolution:** Fix network path or use internal mirror.

### Troubleshooting 5 - TLS Certificate Issue

**Symptoms:** Certificate verification error.

**Root Cause:** Internal registry certificate not trusted.

**Investigation:** Check registry certificate and trust configuration.

**Resolution:** Install proper CA certificate or configure trusted registry path.

### Troubleshooting 6 - Image In Use

**Symptoms:** `podman image rm` fails.

**Root Cause:** A container still references the image.

**Commands:**

```bash
podman ps --all
podman rm -f CONTAINER
podman image rm IMAGE
```

**Validation:** Image is removed.

### Troubleshooting 7 - Push Permission Denied

**Symptoms:** `podman push` fails.

**Root Cause:** User cannot push to repository.

**Commands:**

```bash
podman login REGISTRY
podman push REGISTRY/NAMESPACE/IMAGE:TAG
```

**Resolution:** Create repository or fix access rights.

### Troubleshooting 8 - Build Fails

**Symptoms:** `podman build` stops at a build step.

**Root Cause:** Bad Containerfile, missing base image, or failing command.

**Commands:**

```bash
podman build -f Containerfile -t test-image .
```

**Resolution:** Fix the failing instruction.

### Troubleshooting 9 - Local Storage Full

**Symptoms:** Pull or build fails due to space.

**Root Cause:** Too many unused images.

**Commands:**

```bash
podman system df
podman image prune -af
```

**Resolution:** Remove unused images and containers.

### Troubleshooting 10 - Unqualified Image Pulls Wrong Source

**Symptoms:** Wrong image is pulled.

**Root Cause:** Registry search order.

**Commands:**

```bash
cat /etc/containers/registries.conf
podman pull FULLY_QUALIFIED_IMAGE
```

**Resolution:** Use fully qualified image names.

## Scenario-Based Labs

### Scenario 1 - Kubernetes Cannot Pull Image

**Problem:** Deployment fails with image pull error.

**Likely Causes:** Wrong tag, private registry without secret, registry outage.

**Podman Investigation:**

```bash
podman pull REGISTRY/NAMESPACE/IMAGE:TAG
podman login REGISTRY
```

**Solution:** Fix image name, credentials, or registry access.

### Scenario 2 - Wrong Image Deployed in Production

**Problem:** Production app runs unexpected code.

**Root Cause:** Mutable tag reused.

**Solution:** Use immutable tags such as release version or Git SHA.

### Scenario 3 - Security Team Finds Vulnerable Base Image

**Problem:** Image contains vulnerable packages.

**Solution Workflow:**

```bash
podman pull UPDATED_BASE_IMAGE
podman build -f Containerfile -t app:patched
podman push REGISTRY/ORG/app:patched
```

### Scenario 4 - Registry Access Failure in CI/CD

**Problem:** Pipeline cannot push image.

**Root Cause:** Expired token or missing repository permission.

**Solution:** Rotate credentials and validate with `podman login` and `podman push`.

### Scenario 5 - Developer Used Public Image Directly

**Problem:** Production manifest references unapproved public registry.

**Solution:** Mirror approved image into internal registry and update manifests.

# Section 7: Interview Preparation

## Beginner Questions

### Question 1

**Question:** What is a container image registry?

**Answer:** It is a service used to store and distribute container images.

**Explanation:** Podman, OpenShift, and Kubernetes pull images from registries.

**Follow-up:** Name two registries.

**What Interviewer Checks:** Basic container ecosystem understanding.

### Question 2

**Question:** What is an image tag?

**Answer:** A tag is a label that identifies a version or variant of an image.

**Follow-up:** Why is `latest` risky?

**What Interviewer Checks:** Release discipline.

## Intermediate Questions

### Question 3

**Question:** How do you inspect the default command of an image?

**Answer:**

```bash
podman image inspect IMAGE --format="{{.Config.Cmd}}"
```

**Follow-up:** What other metadata is useful?

**What Interviewer Checks:** Troubleshooting ability.

### Question 4

**Question:** Why should images use semantic versioning?

**Answer:** It communicates major, minor, and patch changes clearly and helps teams manage compatibility.

**Follow-up:** What tag would you use for a production release?

**What Interviewer Checks:** Production readiness.

## Advanced Questions

### Question 5

**Question:** How do you design an enterprise image promotion workflow?

**Answer:** Build once, scan, sign, push to registry, promote the same image through dev, test, staging, and production by tag or digest.

**Follow-up:** Why build once?

**What Interviewer Checks:** CI/CD maturity.

### Question 6

**Question:** What happens if multiple tags point to the same image ID?

**Answer:** Removing one tag may not remove the image layers because another tag still references the same image.

**Follow-up:** How do you fully remove it?

**What Interviewer Checks:** Image storage understanding.

## Scenario-Based Questions

### Question 7

**Question:** A Kubernetes deployment shows `ImagePullBackOff`. How do you investigate?

**Answer:** Check image name, tag, registry availability, authentication, pull secret, and permissions. Try pulling the image manually with Podman.

**Follow-up:** What if manual pull works but Kubernetes pull fails?

**What Interviewer Checks:** Cluster troubleshooting thinking.

### Question 8

**Question:** A production app changed even though no deployment manifest changed. What might have happened?

**Answer:** The manifest might use a mutable tag like `latest`, and the registry tag now points to a new image.

**Follow-up:** How do you prevent it?

**What Interviewer Checks:** Understanding of immutable releases.

# Section 8: Certification Preparation

## Red Hat Exam Tips

- Know image name structure.
- Practice `podman pull`, `podman image ls`, `podman image inspect`, `podman image tag`, `podman image rm`, and `podman image prune`.
- Do not forget container dependency when removing images.
- Know that `latest` is default when no tag is specified.
- Know how to run an image to verify it works.
- Practice with Quay image references.

## Frequently Tested Concepts

- Registries
- Tags
- Pulling images
- Listing images
- Tagging images
- Building images
- Inspecting image metadata
- Removing images
- Pruning images
- Registry authentication

## Common Mistakes

- Forgetting the tag
- Using wrong registry path
- Trying to remove an image used by a container
- Confusing `podman image rm` with `podman rm`
- Forgetting to login before push
- Using `latest` without realizing it

## Top 50 Certification Questions

1. What command pulls an image?
2. What command lists local images?
3. What command searches registries?
4. What command authenticates to a registry?
5. What command builds an image from a Containerfile?
6. What command tags an image?
7. What command pushes an image?
8. What command inspects image metadata?
9. What command removes an image?
10. What command removes dangling images?
11. What does `latest` mean?
12. Why is `latest` risky?
13. What is a registry?
14. What is Quay.io?
15. What is Red Hat Catalog?
16. What is UBI?
17. What is the format of an image reference?
18. What is a namespace in an image reference?
19. What happens if no tag is specified?
20. Where are user-pulled rootless images stored conceptually?
21. Why might image removal fail?
22. How do you force image removal?
23. How do you remove all unused images?
24. How do you inspect an image command?
25. How do you inspect exposed ports?
26. How do you test a pulled image?
27. How do you tag an image for Quay?
28. Why should production use specific tags?
29. Why use trusted registries?
30. What is `registries.conf` used for?
31. What is an unqualified image name?
32. How do you block a registry conceptually?
33. Why does registry authentication matter?
34. What is a base image?
35. What is a Containerfile?
36. What does `FROM` do?
37. What does `CMD` do?
38. How do you build with a custom Containerfile path?
39. How do you list only image repositories?
40. How do you remove a single tag?
41. How do you verify two tags point to the same image?
42. What is semantic versioning?
43. What does MAJOR version mean?
44. What does MINOR version mean?
45. What does PATCH version mean?
46. Why scan images?
47. What is a private registry?
48. What is a public registry?
49. How does Podman know where to search?
50. What should you do before pushing to a registry?

## Command-Based Certification Practice

```bash
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman image ls
podman image inspect registry.access.redhat.com/ubi8/python-39:latest --format="{{.Config.Cmd}}"
podman image tag registry.access.redhat.com/ubi8/python-39:latest python-app:1.0
podman image rm python-app:1.0
podman image prune -af
```

# Section 9: Career Perspective

| Role | How Chapter 3 Is Used |
| --- | --- |
| DevOps Engineer | Builds, tags, pushes, and promotes images in CI/CD pipelines |
| Platform Engineer | Defines approved base images and registry policies |
| Cloud Engineer | Integrates cloud registries such as ECR, ACR, or Artifact Registry |
| SRE | Troubleshoots image pull failures and deployment rollouts |
| OpenShift Administrator | Manages image pull secrets, internal registry, and trusted image sources |
| Kubernetes Administrator | Fixes `ImagePullBackOff`, registry auth, and image tag issues |
| DevSecOps Engineer | Scans, signs, verifies, and controls image supply chain risk |

# Section 10: Beyond the Chapter

## What to Learn Next

- Containerfile best practices
- Multi-stage builds
- Image scanning
- Image signing
- SBOMs
- Skopeo
- Buildah
- Red Hat Quay
- OpenShift image streams
- Kubernetes image pull secrets
- GitOps image promotion

## Related Tools

| Tool | Purpose |
| --- | --- |
| Skopeo | Inspect and copy remote images without pulling |
| Buildah | Build OCI images |
| Quay | Enterprise registry |
| Clair | Vulnerability scanning engine often associated with Quay |
| Cosign | Image signing and verification |
| Trivy | Vulnerability scanning |
| Syft | SBOM generation |
| Grype | Vulnerability scanning from SBOM/image |

## Connection to Platform Engineering

Container images are the artifact that moves from developer laptop to CI/CD to registry to Kubernetes or OpenShift. A strong image workflow is the foundation for secure and repeatable platform delivery.

# Section 11: Final Mastery Guide

## 1. Complete Chapter Summary

Chapter 3 teaches how to work with container images as shareable deployment artifacts. The chapter introduces registries, Red Hat Catalog, Quay.io, UBI images, registry configuration, authentication, image tags, semantic versioning, image pulling, building, pushing, inspecting, removing, and pruning.

## 2. One-Page Revision Sheet

- Registry stores images.
- Red Hat Catalog helps find trusted Red Hat images.
- Quay stores user and organization images.
- UBI is a trusted Red Hat base image family.
- Image reference format is `registry/namespace/name:tag`.
- If no tag is used, Podman assumes `latest`.
- Avoid `latest` in production.
- Use `podman login` for protected registries.
- Use `podman pull` to download images.
- Use `podman image ls` to list images.
- Use `podman build` to build images.
- Use `podman image tag` to add tags.
- Use `podman push` to upload images.
- Use `podman image inspect` to view metadata.
- Use `podman image rm` to remove images.
- Use `podman image prune` to clean unused images.

## 3. Command Cheat Sheet

```bash
podman login quay.io
podman search python
podman pull registry.access.redhat.com/ubi8/python-39:latest
podman image ls
podman build -f Containerfile -t app:1.0
podman image tag app:1.0 quay.io/YOUR_USER/app:1.0
podman push quay.io/YOUR_USER/app:1.0
podman image inspect app:1.0
podman image inspect app:1.0 --format="{{.Config.Cmd}}"
podman image rm app:1.0
podman image prune -af
```

## 4. Security Cheat Sheet

- Use trusted registries.
- Avoid unqualified image names.
- Avoid `latest`.
- Use registry authentication.
- Use least privilege credentials.
- Scan images.
- Keep base images patched.
- Do not store secrets in images.
- Use approved base images.
- Use image signing where required.

## 5. Troubleshooting Cheat Sheet

| Problem | Check |
| --- | --- |
| Image not found | Registry, namespace, image name, tag |
| Unauthorized | `podman login`, credentials, permissions |
| Wrong version | Tag, image ID, digest |
| Cannot remove image | Containers using the image |
| Push denied | Repository exists, user has push access |
| Pull timeout | DNS, firewall, proxy, registry availability |
| Build fails | Containerfile syntax and base image access |

## 6. Production Best Practices Checklist

- [ ] Use fully qualified image names.
- [ ] Use specific version tags.
- [ ] Avoid `latest`.
- [ ] Use approved base images.
- [ ] Scan images.
- [ ] Authenticate with service accounts.
- [ ] Push only to approved registries.
- [ ] Keep image metadata traceable.
- [ ] Clean unused images.
- [ ] Document image ownership.

## 7. DevOps Engineer Notes

Build once, tag properly, scan, push, and promote the same artifact through environments.

## 8. Platform Engineer Notes

Create golden base images, define registry policies, and standardize image workflows.

## 9. Cloud Engineer Notes

Understand how Podman image workflows map to ECR, ACR, Artifact Registry, and private cloud registries.

## 10. SRE Notes

Know how to debug pull failures, tag mistakes, registry outages, and storage pressure.

## 11. OpenShift Engineer Notes

Connect this chapter to image streams, image pull secrets, internal registry, and trusted image policies.

## 12. Kubernetes Engineer Notes

Connect this chapter to deployment image fields, pull policies, secrets, and `ImagePullBackOff`.

## 13. DevSecOps Notes

Focus on scanning, signing, approved registries, base image governance, and supply chain controls.

## 14. Top 20 Takeaways

1. Images are deployment artifacts.
2. Registries store and distribute images.
3. Use trusted registries.
4. Red Hat Catalog helps inspect Red Hat images.
5. UBI images are important Red Hat base images.
6. Quay is used to store custom images.
7. Tags identify versions.
8. `latest` is risky.
9. One image can have multiple tags.
10. Pull images with `podman pull`.
11. List images with `podman image ls`.
12. Build images with `podman build`.
13. Tag images with `podman image tag`.
14. Push images with `podman push`.
15. Inspect images with `podman image inspect`.
16. Remove images with `podman image rm`.
17. Prune images carefully.
18. Login is needed for protected registries.
19. Registry configuration affects unqualified image pulls.
20. Image security is part of DevSecOps.

## 15. 30-Day Practice Plan

| Day | Practice |
| --- | --- |
| 1-3 | Pull, list, inspect images |
| 4-6 | Practice image tags |
| 7-9 | Build simple images |
| 10-12 | Push and pull from Quay |
| 13-15 | Practice cleanup and prune |
| 16-18 | Troubleshoot pull and auth failures |
| 19-21 | Build CI/CD style image workflow |
| 22-24 | Practice security checks and metadata audit |
| 25-27 | Simulate Kubernetes image pull failures |
| 28-30 | Create final portfolio project and README |

## 16. Recommended Hands-On Projects

- Build and push a personal portfolio web image.
- Create a Python runtime image and tag it semantically.
- Push an image to Quay and pull it on another machine.
- Create a registry troubleshooting checklist.
- Build a CI/CD-style image promotion demo.

## 17. Recommended Home Lab Setup

- RHEL or Fedora VM
- Podman
- Quay.io account
- GitHub repository for notes
- Optional local registry
- Optional OpenShift Local or Kubernetes cluster

## LinkedIn Post Draft

Today I completed Day 03 of my DO188 preparation: Chapter 3 - Container Images.

I practiced how container images are stored, pulled, tagged, inspected, built, pushed, and removed with Podman. I also studied Red Hat registries, Red Hat Catalog, Quay.io, UBI images, registry authentication, image versioning, and why using specific tags is important for production.

Key commands practiced:

- `podman pull`
- `podman image ls`
- `podman search`
- `podman login`
- `podman build`
- `podman image tag`
- `podman push`
- `podman image inspect`
- `podman image rm`
- `podman image prune`

The biggest lesson: container images are not just files. They are release artifacts, and real DevOps teams must manage them with versioning, security, trust, and repeatability.

#RedHat #DO188 #Podman #Containers #OpenShift #DevOps #PlatformEngineering #SRE #DevSecOps

## GitHub Commit Message

```text
Add Chapter 3 container images study guide
```
