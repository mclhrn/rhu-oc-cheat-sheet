## Summary
 * Applications typically log activity, such as events, warnings and errors, to aid the analysis of application behavior.
 * Container applications should print log data to standard output, instead of to a file, to enable easy access to logs.
 * To review the logs for a container deployed locally with Podman, use the podman logs command.
 * Use the oc logs command to access logs for BuildConfig and Deployment objects, as well as individual pods within an OpenShift project.
 * The -f option allows you to monitor the log output in near real-time for both the podman logs andoc logscommands.
 * Use the oc port-forward command to connect directly to a port on an application pod. You should only leverage this technique on non-production pods, because interactions can alter the behavior of the pod.

```
# logs for bc
oc logs bc/<application-name>

# troubleshoot failed builds
oc start-build <application-name>

# troubleshoot failed deployments
oc logs deployment/<application-name>

# relax project security
oc adm policy add-scc-to-user anyuid -z default

# port forward
oc port-forward db 30306 3306

# remote debugging must be set at server startup for that service
JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,address=8787,server=y,suspend=n"

# get events
oc get events

# access running containers
oc exec [options] pod [-c container] -- command [arguments]
oc exec -it myhttpdpod /bin/bash

# override container binaries
podman run -it -v /bin:/bin image /bin/bash

# alternatively install binaries at build step
RUN yum install -y \
      less \
      dig \
      ping \
      iputils && \
    yum clean all

# transfer files via
# volume mounts
podman run -v /conf:/etc/httpd/conf -d do180/apache

# copy files from host to running container
podman cp standalone.conf todoapi:/opt/jboss/standalone/conf/standalone.conf

# copy files from container to host
podman cp todoapi:/opt/jboss/standalone/conf/standalone.conf .

# pipe cmds that are executed in conatiner
podman exec -i <container> mysql -uroot -proot < /path/on/host/db.sql

# the opposite ^
podman exec -it <containerName> sh \
    -c 'exec mysqldump -h"$MYSQL_PORT_3306_TCP_ADDR" \
    -P"$MYSQL_PORT_3306_TCP_PORT" \
    -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" items' > db_dump.sql

# new app
oc new-app \
    --name nodejs-dev \
    https://github.com/${RHT_OCP4_GITHUB_USER}/DO180-apps#troubleshoot-review \
    -i nodejs:16-ubi8 --context-dir=nodejs-app --build-env \
    npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs

```