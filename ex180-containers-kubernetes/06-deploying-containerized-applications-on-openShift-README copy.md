### This section is light due to most of it being basic working knowledge/experience

## Summary
 * OpenShift Container Platform stores definitions of each OpenShift or Kubernetes resource instance as an object in the cluster's distributed database service, etcd. Common resource typesare:Pod,Persistent Volume(PV),Persistent Volume Claim(PVC),Service (SVC),Route,Deployment,DeploymentConfigandBuild Configuration(BC).
 * Use the OpenShift command-line client oc to:
   * Create, change, and delete projects.
   * Create application resources inside a project.
   * Delete, inspect, edit, and export resources inside a project.
   * Check logs from application pods, deployments, and build operations.
 * The oc new-app command can create application pods in many different ways: from an existing container image hosted on an image registry, from Containerfiles, and from source code using the Source-to-Image (S2I) process.
 * Source-to-Image (S2I) is a tool that makes it easy to build a container image from application source code. This tool retrieves source code from a Git repository, injects the source code into a selected container image based on a specific language or technology, and produces a new container image that runs the assembled application.
 * A Route connects a public-facing IP address and DNS host name to an internal-facing service IP. While services allow for network access between pods inside an OpenShift instance, routes allow for network access to pods from users and applications outside the OpenShift instance.
 * You can create, build, deploy, and monitor applications using the OpenShift web console.

```
# new-app s2i
oc new-app php:7.3 --name=php-helloworld https://github.com/${RHT_OCP4_GITHUB_USER}/DO180-apps#s2i --context-dir php-helloworld

# logs
oc logs --all-containers -f php-helloworld-1-build

# describe deployment
oc describe deployment/php-helloworld

# add route
oc expose service php-helloworld --name ${RHT_OCP4_DEV_USER}-helloworld

# find route
oc get route -o jsonpath='{..spec.host}{"\n"}'

# Explore starting application builds by changing the application in its Git repository and executing the proper commands to start a new Source-to-Image build.
1. Edit files
2. Commit and push to repo
3. oc start-build php-helloworld
```
