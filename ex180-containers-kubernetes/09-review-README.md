Task:

1. Create a container image that starts an instance of a Nexus server:
   * The/home/student/DO180/labs/comprehensive-review/imagedirectory contains files for building the container image. Execute the get-nexus-bundle.sh script to retrieve the Nexus server files.
   * Write a Containerfile that containerizes the Nexus server. The Containerfile must be located in the /home/student/DO180/labs/comprehensive-review/image directory. The Containerfile must also:
     * Use a base image of ubi8/ubi:8.5 and set an arbitrary maintainer.
     * Set the ARG variable NEXUS_VERSION to 2.14.3-02, and set the environment variable
NEXUS_HOME to /opt/nexus.
     * Install the java-1.8.0-openjdk-devel package.
     * Run a command to create a nexus user and group. They both have a UID and GID of 1001. 
     * Change the permissions of the ${NEXUS_HOME}/ directory to 775.
     * Unpack the nexus-2.14.3-02-bundle.tar.gz file to the ${NEXUS_HOME}/directory. Add the nexus-start.sh to the same directory.
     * Run a command, ln -s ${NEXUS_HOME}/nexus-${NEXUS_VERSION} ${NEXUS_HOME}/nexus2, to create a symlink in the container. Run a command to recursively change the ownership of the Nexus home directory to nexus:nexus.
     * Make the container run as the nexus user, and set the working directory to /opt/ nexus.
     * Define a volume mount point for the /opt/nexus/sonatype-work container directory. The Nexus server stores data in this directory.
     * Set the default container command to nexus-start.sh.
   * Build the container image with the name nexus.

2. Build and test the container image using Podman with a volume mount:
   * Usethescript/home/student/DO180/labs/comprehensive-review/deploy/ local/run-persistent.sh to start a new container with a volume mount.
   * Review the container logs to verify that the server is started and running.
   * Test access to the container service using the URL: http://127.0.0.1:18081/nexus.
   * Remove the test container.

3. Deploy the Nexus server container image to the OpenShift cluster. You must:
   * Tag the Nexus server container image as quay.io/${RHT_OCP4_QUAY_USER}/ nexus:latest, and push it corresponding public repository on quay.io.
   * Create an OpenShift project with a name of ${RHT_OCP4_DEV_USER}-review.
   * Edit the deploy/openshift/resources/nexus-deployment.yaml and replace RHT_OCP4_QUAY_USER with your Quay username. Create the Kubernetes resources.
   * Create a route for the Nexus service. Verify that you can access http://nexus- ${RHT_OCP4_DEV_USER}-review.${RHT_OCP4_WILDCARD_DOMAIN}/nexus/ from workstation.

```
# completed containerfile

FROM ubi8/ubi:8.5

MAINTAINER username <username@example.com>

ARG NEXUS_VERSION=2.14.3-02
ENV NEXUS_HOME=/opt/nexus

RUN yum install -y --setopt=tsflags=nodocs java-1.8.0-openjdk-devel \
    && yum clean all -y
RUN groupadd -r nexus -f -g 1001 && \
    useradd -u 1001 -r -g nexus -m -d ${NEXUS_HOME} -s /sbin/nologin -c "Nexus User" nexus && \
    chown -R nexus:nexus ${NEXUS_HOME} && \
    chmod -R 755 ${NEXUS_HOME}

USER nexus

ADD nexus-${NEXUS_VERSION}-bundle.tar.gz ${NEXUS_HOME}
ADD nexus-start.sh ${NEXUS_HOME}/

RUN ln -s ${NEXUS_HOME}/nexus-${NEXUS_VERSION} ${NEXUS_HOME}/nexus2

WORKDIR ${NEXUS_HOME}

VOLUME ["/opt/nexus/sonatype-work"]

CMD ["sh", "nexus-start.sh"]

# build the image
podman build --layers=false -t nexus .

# publish to quay
podman push localhost/nexus:latest quay.io/${RHT_OCP4_QUAY_USER}/nexus:latest 
```
