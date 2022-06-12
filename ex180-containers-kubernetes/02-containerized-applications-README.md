## Summary
* Containers are isolated application runtimes, created with very little overhead.
* A container image packages an application with all of its dependencies, making it easier to run the application in different environments.
* Applications such as Podman create containers using features of the standard Linux kernel.
* Container image registries are the preferred mechanism for distributing container images to multiple users and hosts.
* OpenShift orchestrates applications composed of multiple containers using Kubernetes.
* Kubernetes manages load balancing, high availability, and persistent storage for containerized applications.
* OpenShift adds to Kubernetes multitenancy, security, ease of use, and continuous integration and continuous development features.
* OpenShift routes enable external access to containerized applications in a manageable way.

```
# search a registry
podman search rhel

# search a specific registry
podman search registry.redhat.io/rhel

# log in to registry
podman login registry.redhat.io

# pull image
podman pull registry.redhat.io/rhel7-atomic

# run container
podman run 

# run pass in env vars
podman run -e GREET=Hello -e NAME=RedHat ubi7/ubi:8.3 printenv GREET NAME

# run in background
podman run -d -p 8080 registry.redhat.io/rhel8/httpd-24

# run in bg with port and name
podman run -d -p 8080:80 --name httpd-basic quay.io/redhattraining/httpd-parent:2.4

# view local images
podman images

# open interactive session
podman run -it registry.redhat.io/ubi7 /bin/bash

# open interactive session as root
podman exec -it httpd-basic /bin/bash
```
