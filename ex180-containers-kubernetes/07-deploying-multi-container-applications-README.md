## Summary
 * Software-defined networks enable communication between containers. Containers must be attached to the same software-defined network to communicate.
 * Containerized applications cannot rely on fixed IP addresses or host names to find services.
 * Podman uses Container Network Interface (CNI) to create a software-defined network and attaches all containers on the host to that network. Kubernetes and OpenShift create a software-defined network between all containers in a pod.
 * Within the same project, Kubernetes injects a set of variables for each service into all pods.
 * OpenShift templates automate creating applications consisting of multiple resources. Template parameters allow using the same values when creating multiple resources.

```
# build image from containerfile
cd images/mysql
podman build -t do180-mysql-80-rhel8 .

# push it to quay
podman login quay.io -u ${RHT_OCP4_QUAY_USER}
podman tag do180-mysql-80-rhel8 quay.io/${RHT_OCP4_QUAY_USER}/do180-mysql-80-rhel8
podman push quay.io/${RHT_OCP4_QUAY_USER}/do180-mysql-80-rhel8
cd ~

# upload template for others to use
oc create -f quote-php-template.json

# deploy template
oc process quote-php-persistent \
  -p RHT_OCP4_QUAY_USER=${RHT_OCP4_QUAY_USER} \
  | oc create -f -

#v expose service
oc expose svc quote-php
```
