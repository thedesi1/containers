# containers

K8s tailored container images for home lab to learn and build cloud native apps.

## Why? Really why we need this?

The goal of this project is to support [semantically versioned](https://semver.org/), [rootless](https://rootlesscontaine.rs/), and [multiple architecture](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/) containers for various applications.

We aspire to achive, if possible, logging to stdout, [one process per container](https://testdriven.io/tips/59de3279-4a2d-4556-9cd0-b444249ed31e/), no [s6-overlay](https://github.com/just-containers/s6-overlay) and all images are built on top of [Alpine](https://hub.docker.com/_/alpine) or [Ubuntu](https://hub.docker.com/_/ubuntu).

## Using GitHub Actions to Build Docker Images
While Docker is selected as the container runtime, there are plenty of other container runtimes to use as well such as Containerd and Podman. The image specification to build a container has been standardized as the [OCI Image Format](https://github.com/opencontainers/image-spec "OCI Image Format"). Docker is one of the most popular and has excellent cross-platform support, easier to use, which helps others who are new to containers. 
### The Dockerfile
Dockerfile contains the instructions to build an image from any source code. A Dockerfile at a 10,000, foot view is a set of instructions to build my application and package it up as an image.

#### Dockerfile Best Practices

`Dockerfile` practices that is followed:

1.  Use a `.dockerignore` file to exclude unnecessary files and directories to increase the build’s performance.
2.  Use **trusted base images** only and keep updating the images periodically.
3.  Each instruction in the `Dockerfile` adds an extra layer to the Docker image. Minimize the number of layers by consolidating the instructions to increase the build’s performance and time.
4.  Run as a **Non-Root User** to avoid security breaches.
5.  **Keep the image small:** Reduce the image size for faster deployment and avoid installing unnecessary tools in your image. Use minimal images to reduce the attack surface.
6.  Use specific tags over the latest tag for the image to avoid breaking changes over time.
7.  Avoid using multiple `RUN` commands as it creates multiple cacheable layers which will affect the efficiency of the build process.
8.  Never share or copy the application credentials or any sensitive information in the `Dockerfile`. If you use it, add it to .`dockerignore`
9.  Use `EXPOSE` and `ENV` commands as late as possible in `Dockerfile`.
10.  **Use a linter:** Use a linter like hadolint to check your Dockerfile for common issues and best practices.
11.  **Use a single process per container:** Each container should run a single process. This makes it easier to manage and monitor containers, and helps to keep containers lightweight.
12.  **Use multi-stage builds:** Use multi-stage builds to create smaller and more efficient images.

### Tag immutability

The containers built here do not use immutable tags, as least not in the more common way you have seen from [linuxserver.io](https://fleet.linuxserver.io/) or [Bitnami](https://bitnami.com/stacks/containers). 

We take do take a similar approach but instead of appending a `-ls69` or `-r420` prefix to the tag we instead insist on pinning to the sha256 digest of the image, while this is not as pretty it is just as functional in making the images immutable.

| Container                                          | Immutable |
|----------------------------------------------------|-----------|
| `ghcr.io/thedesi1/alpine:rolling`                   | ❌         |
| `ghcr.io/thedesi1/alpine:3.17.3`                | ❌         |
| `ghcr.io/thedesi1/alpine:rolling@sha256:8053...`    | ✅         |
| `ghcr.io/thedesi1/alpine:3.17.3@sha256:8053...` | ✅         |

_If pinning an image to the sha256 digest, tools like [Renovate](https://github.com/renovatebot/renovate) support updating the container on a digest or application version change._

### Passing arguments to a application

Some applications do not support defining configuration via environment variables and instead only allow certain config to be set in the command line arguments for the app. To circumvent this, for applications that have an `entrypoint.sh` read below.

1. First read the Kubernetes docs on [defining command and arguments for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/).
2. Look up the documentation for the application and find a argument you would like to set.
3. Set the argument in the `args` section, be sure to include `entrypoint.sh` as the first arg and any application specific arguments thereafter.

    ```yaml
    args:
      - /entrypoint.sh
      - --port
      - "8080"
    ```

### Configuration volume

For applications that need to have persistent configuration data the config volume is hardcoded to `/config` inside the container. This is not able to be changed in most cases.
