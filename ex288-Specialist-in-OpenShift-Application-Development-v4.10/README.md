## Introduction
Red Hat OpenShift Development II: Building Kubernetes Applications
Red Hat OpenShift Container Platform, which is based on container technology and Kubernetes, provides developers with an enterprise-ready solution for developing and deploying containerized software applications.

Red Hat OpenShift Development II: Building Kubernetes Applications (DO288), the second course in the OpenShift development track, teaches students how to design, build, and deploy containerized software applications on an OpenShift cluster. Whether writing native container applications or migrating existing applications, this course provides hands-on training to boost developer productivity powered by Red Hat OpenShift Container Platform.

### Course Objectives
Design, build, and deploy containerized applications on an OpenShift cluster.

### Chapter 1. Deploying and Managing Applications on an OpenShift Cluster
#### Objectives
* Describe the architecture and new features in OpenShift 4.
* Deploy an application to the cluster from a Dockerfile with the CLI.
* Deploy an application from a container image and manage its resources using the web console.
* Deploy an application from source code and manage its resources using the command-line interface.

```shell
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}

oc new-project ${RHT_OCP4_DEV_USER}-name-of-project

# Create a new application from sources in a Git repository. Name the application greet.                                <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
# Use the --build-env option with the oc new-app command to define the build environment variable with the npm modules URL.
oc new-app \                                                                            
    --name greet \
    --build-env npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs \
    nodejs:16-ubi8~https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps#source-build \
    --context-dir nodejs-helloworld

# Inspect logs
oc logs -f bc/greet

# Use the python3 -m json.tool command to validate the JSON (if JSON is the error)                                      <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
python3 -m json.tool nodejs-helloworld/package.json

# Fix errors and push
cd nodejs-helloworld
git commit -a -m 'Fixed JSON syntax'
git push

# Start new build
oc start-build --follow bc/greet

# Expose route
oc expose svc/greet

# Get route
oc get route

# Test route
curl http://greet-${RHT_OCP4_DEV_USER}-source-build.${RHT_OCP4_WILDCARD_DOMAIN}
```

### Chapter 2. Designing Containerized Applications for OpenShift
#### Objectives
* Select an appropriate application containerization method.
* Build a container image with advanced Dockerfile directives.
* Select a method for injecting configuration data into an application and create the necessary resources to do so.

```shell
# Create branch to push and build from
git checkout -b design-container
git push -u origin design-container

# Build the app
podman build --layers=false -t do288-hello-java ./hello-java

# Tag  image                                                                                                            <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
podman tag do288-hello-java \
  quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java

# Login to quay
podman login quay.io -u ${RHT_OCP4_QUAY_USER}

# Push image to quay
podman push --format v2s1 quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java

# Use the image in the repository to create a new application in OpenShift named elvis                                  <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
oc new-app --name elvis quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java                                                  

# Look for errors in deployment/pod
oc get pods
oc logs elvis-XXX

# Update containerfile to have correct permissions                                                                      <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
# Remove any user defined in file 'useradd wildfly && \'
# Replace 
RUN   chown -R wildfly:wildfly /opt/app-root && \
      chmod -R 700 /opt/app-root
      
# With this
RUN   chgrp -R 0 /opt/app-root && \
      chmod -R g=u /opt/app-root

# This changes the group and assigns the same permissions as the user
# Commit changes to git

# Start a new build for the application:
podman rmi -a --force
podman build --layers=false -t do288-hello-java ./hello-java

# Tag the new image and push it to Quay.io
podman tag do288-hello-java quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java
podman push --format v2s1  quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java

# Remove old project and recreate
oc delete project ${RHT_OCP4_DEV_USER}-design-container
oc new-project ${RHT_OCP4_DEV_USER}-design-container
oc new-app  --name elvis quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java
oc get pods
oc logs elvis-XXX

# Expose the service
oc expose svc/elvis
oc get route
curl http://elvis-${RHT_OCP4_DEV_USER}-design-container.${RHT_OCP4_WILDCARD_DOMAIN}/api/hello

# Create a new config map
oc create cm appconfig --from-literal APP_MSG="Elvis lives"
oc describe cm/appconfig

# Add the cm to the dc                                                                                                  <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
oc set env deployment/elvis --from cm/appconfig

# Verify that the new variable is set inside the container                                                              <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
oc rsh elvis-66c7f6d47f-ll2jq env | grep APP_MSG
```

### Chapter 3. Publishing Enterprise Container Images
In this lab, you will publish an OCI-formatted container image to an external registry and deploy an application from that image using an image stream.
#### Objectives
* Manage container images in registries using Linux container tools.
* Access the OpenShift internal registry using Linux container tools.
* Create image streams for container images in external registries.
```shell
# From directory with OCI image run below to copy to external registry                                                  <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
skopeo copy --format v2s1 \
  oci:/home/student/DO288/labs/expose-image/php-info \
  docker://quay.io/${RHT_OCP4_QUAY_USER}/php-info

# Inspect it
skopeo inspect \
  docker://quay.io/${RHT_OCP4_QUAY_USER}/php-info

# Create the imagestream from this image to openshift common project
oc new-project ${RHT_OCP4_DEV_USER}-common

# Create a secret from the container registry API access token that was stored by Podman.                               <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
oc create secret generic quayio \
  --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
  --type kubernetes.io/dockerconfigjson

# Import the image                                                                                                      <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
oc import-image php-info \
  --confirm \
  --reference-policy local \
  --from quay.io/${RHT_OCP4_QUAY_USER}/php-info

# Verify
oc get istag

# Create an app by deploying the container image from this imagestream
oc new-project ${RHT_OCP4_DEV_USER}-expose-image

# Grant service accounts from the new youruser-expose-image project access to                                           <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
# image streams from the youruser-common project.        
oc policy add-role-to-group \
  -n ${RHT_OCP4_DEV_USER}-common system:image-puller \
  system:serviceaccounts:${RHT_OCP4_DEV_USER}-expose-image

# Deploy from common
oc new-app --name info -i ${RHT_OCP4_DEV_USER}-common/php-info
```

### Chapter 4. Managing Builds on OpenShift
#### Objectives
* Describe the OpenShift build process.
* Manage application builds using the BuildConfig resource and CLI commands.
* Trigger the build process with supported methods.
* Process post build logic with a post-commit build hook.

```shell
# Deploy app with error in env var
oc new-app \
    --name simple \
    --build-env npm_config_registry=http://${invalid-server}:8081/repository/nodejs \
    https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps \
    --context-dir build-app

# Fix
oc set env bc simple --list
oc set env bc simple \
  npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs
oc start-build simple

# Use a webhook to start a build
oc describe bc simple
SECRET=$(oc get bc simple -o jsonpath="{.spec.triggers[*].generic.secret}{'\n'}")

curl -X POST -k \
  ${RHT_OCP4_MASTER_API}apis/build.openshift.io/v1/namespaces/${RHT_OCP4_DEV_USER}-build-app/buildconfigs/simple/webhooks/$SECRET/generic

# Other useful commands for triggers and secrets
skopeo copy --format v2s1 \
  oci-archive:php-73-ubi8-original.tar.gz \
  docker://quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8:latest

oc create secret generic quay-registry \
  --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
  --type kubernetes.io/dockerconfigjson

oc secrets link builder quay-registry
```

### Chapter 5. Customizing Source-to-Image Builds
#### Objectives
* Describe the required and optional steps in the Source-to-Image build process.
* Customize an existing S2I builder image with scripts.
* Create a new S2I builder image with S2I tools.

```shell
# Registry url
oc new-app registry.access.redhat.com/ubi8/ubi:8.0

# Git repo
oc new-app https://github.com/RedHatTraining/DO288-apps/ubi-echo

# Force imagestream builder image
oc new-app -i php:7.3 \
  https://github.com/RedHatTraining/DO288-apps/php-helloworld

# Create custom s2i builder image
s2i version

# Create the templatre files for builder image
s2i create <imageName> <destination>

# <destination>
# ├── Dockerfile
# ├── Makefile
# ├── README.md
# ├── s2i
# │   └── bin
# │       ├── assemble
# │       ├── run
# │       ├── save-artifacts
# │       └── usage
# └── test
#     ├── run
#     └── test-app
#         └── index.html
# 4 directories, 9 files

# The additional steps will involve overriding the generated Dockerfile, and the scripts
# located at ~/<destination>/s2i/bin/

podman build -t s2i-do288-httpd .

# Create a new directory for the Containerfile generated by the s2i build command
mkdir ~/s2i-sample-app

# Generate 
s2i build test/test-app/ \
  s2i-do288-httpd \
  s2i-sample-app \
  --as-dockerfile ~/s2i-sample-app/Containerfile

# Build
podman build -t s2i-sample-app .
  
podman run --name test -u 1234 \
  -p 8080:8080 -d s2i-sample-app

# Push to quay
skopeo copy --format v2s1 \
  containers-storage:localhost/s2i-do288-httpd \
  docker://quay.io/${RHT_OCP4_QUAY_USER}/s2i-do288-httpd
  
# Create secret
oc create secret generic quayio \
  --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
  --type=kubernetes.io/dockerconfigjson
  
# Link secret
oc secrets link builder quayio

# Create an imagestream
oc import-image s2i-do288-httpd \
  --from quay.io/${RHT_OCP4_QUAY_USER}/s2i-do288-httpd --confirm

oc get is

oc new-app --name hello-s2i \
  s2i-do288-httpd~https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps \
  --context-dir=html-helloworld
```

### Chapter 6. Deploying Multi-container Applications
#### Objectives
* Describe the elements of an OpenShift template.
* Build a multicontainer application using Helm Charts.
* Customize OpenShift deployments.

```shell

# Use helm to create chart
helm create exochart

# Add dependencies to Chart.yaml
dependencies:
- name: cockroachdb
  version: 6.0.4
  repository: https://charts.cockroachdb.com/
  
# Update dependencies. This pulls in the tarball for cockroachdb defined above
helm dependency update

# Add a range to deployment, referencing values.yaml
env:
  {{- range .Values.env }}
- name: "{{ .name }}"
  value: "{{ .value }}"
  {{- end }}

# in values.yaml
env:
  - name: "DB_HOST"
    value: "exoplanets-cockroachdb"
  - name: "DB_NAME"
    value: "postgres"
  - name: "DB_USER"
    value: "root"
  - name: "DB_PORT"
    value: "26257"
    
# Install chart
helm install exoplanets .

# Kustomise
mkdir exokustom
cd exokustom
mkdir base

# use helm template to extract object definitions into base definition
helm template exoplanets ../exochart > base/deployment.yaml

# Create a kustomization.yaml file in base and add deployment.yaml as resource
resources:
- deployment.yaml

# Create and apply the Test overlay in the Kustomize directory, using a directory called test
mkdir -p overlays/test

# Create a replica_limits.yaml in test with test env specific values
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exoplanets-exochart
spec:
  replicas: 5
  template:
    spec:
      containers:
        - name: exochart
          resources:
            limits:
              memory: "128Mi"
              cpu: "250m"
              
# Create the overlays/test/kustomization.yaml file, 
# add the base directory as the base definition, 
# and the replica_limits.yaml file as a patch
bases:
- ../../base
patches:
- replica_limits.yaml

# Apply the Test overlay to the running instance of the Exoplanets application
oc apply -k overlays/test
```

### Chapter 7. Managing Application Deployments
#### Objectives
* Implement liveness and readiness probes.
* Select the appropriate deployment strategy for a cloud-native application.
* Manage the deployment of an application with CLI commands.

```shell
oc set probe deployment probes --liveness \
  --get-url=http://:8080/healthz \
  --initial-delay-seconds=2 --timeout-seconds=2
  
oc set probe deployment probes --readiness \
  --get-url=http://:8080/ready \
  --initial-delay-seconds=2 --timeout-seconds=2
  
oc describe deployment probes | grep -iA 1 liveness
  
# Deployment strategy
# Remove triggers
oc set triggers dc/mysql --from-config --remove

oc patch dc/mysql --patch \
  '{"spec":{"strategy":{"type":"Recreate"}}}'
  
# Remove rollingParams attribute
oc patch dc/mysql --type=json \
  -p='[{"op":"remove", "path": "/spec/strategy/rollingParams"}]'
  
oc patch dc/mysql --patch \
  '{"spec":{"strategy":{"recreateParams":{"post":{"failurePolicy": "Abort","execNewPod":{"containerName":"mysql","command":["/bin/sh","-c","curl -L -s https://github.com/RedHatTraining/DO288-apps/releases/download/OCP-4.1-1/import.sh -o /tmp/import.sh&&chmod 755 /tmp/import.sh&&/tmp/import.sh"]}}}}}}'
  
oc rollback dc/quip
  
oc whoami --show-console  
```

### Chapter 8. Building Applications for OpenShift
#### Objectives
* Integrate a containerized application with non-containerized services.
* Deploy containerized third-party applications following recommended practices for OpenShift.
* Use a Red Hat OpenShift Application Runtime to deploy an application.

```shell
oc create svc externalname tododb \
  --external-name ${RHT_OCP4_MYSQL_SERVER}
  
# Jkube
# Configure the application by using the JKUbe plugin for deployment on OpenShift
<properties>
  <jkube.build.switchToDeployment>true</jkube.build.switchToDeployment>
</properties>


<build>
    <plugins>
      <!-- JKube Maven plugin -->
      <plugin>
        <groupId>org.eclipse.jkube</groupId>1
        <artifactId>openshift-maven-plugin</artifactId>
        <version>1.2.0</version>
      </plugin>

# Create a jkube dir in root of java app
src/main/jkube/

# oc:build performs an s2i
# oc:resource creates several resources
mvn -DskipTests package oc:build oc:resource

# Deploy the app
mvn -DskipTests oc:deploy
```

















