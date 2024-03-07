***************************
* *Version 4.12 Specific* *
***************************

## Introduction
This course prepares a senior OpenShift Cluster Administrator to perform daily administration tasks on clusters that host applications that internal teams and external vendors provide; enable self-service for cluster users with different roles; and deploy applications that require special permissions, such as CI/CD tooling, performance monitoring, and security scanners.
DO280 focuses on configuring multi-tenancy and security features of OpenShift. DO280 also teaches how to manage OpenShift add-ons based on operators. This course is based on Red Hat® OpenShift® Container Platform 4.12.

### Course Objectives
* Configure and manage OpenShift clusters to maintain security and reliability across multiple applications and development teams.
* Configure authentication, authorization, and resource quotas.
* Protect network traffic with network policies and TLS security (HTTPS).
* Expose applications by using protocols other than HTTP and TLS, and attach applications to multi-homed networks.
* Manage OpenShift cluster updates and Kubernetes operator updates.
* This course, together with the Red Hat OpenShift I: Containers & Kubernetes (DO180) course, prepares the student to take the Red Hat Certified Specialist in OpenShift Administration exam (EX280).

### Chapter 1. Declarative Resource Management
#### Objectives
* Deploy and update applications from resource manifests that are stored as YAML files.
* Deploy and update applications from resource manifests that are augmented by Kustomize.

##### Kustomise

In a kustomise project, dir looks like this:
```shell
$ tree
.
├── base
│   ├── database
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   ├── secret.yaml
│   │   └── service.yaml
│   ├── exoplanets
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   ├── route.yaml
│   │   ├── secret.yaml
│   │   └── service.yaml
│   └── kustomization.yaml
├── overlays 
│   └── production
│       ├── kustomization.yaml
│       └── patch-replicas.yaml
└── README.md

3 directories, 16 files

$ cat base/kustomization.yaml
kind: Kustomization
bases:
- database
- exoplanets

$ oc apply -k base

$ cat \
  overlays/production/kustomization.yaml
kind: Kustomization
bases:
- ../../base/
patches:
- path: patch-replicas.yaml
  target:
    kind: Deployment
    name: exoplanets
    
$ cat \
  overlays/production/patch-replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exoplanets
spec:
  replicas: 2   
```

### Chapter 2. Deploy Packaged Applications
#### Objectives
* Deploy an application and its dependencies from resource manifests that are stored in an OpenShift template.
* Deploy and update applications from resource manifests that are packaged as Helm charts.

##### Using Templates

```shell
$ oc new-app --template=cache-service -p APPLICATION_USER=my-user

$ oc process --parameters -f  my-cache-service.yaml

$ oc process my-cache-service \
  -p APPLICATION_USER=user1 -o yaml > my-cache-service-manifest.yaml
  
# Process params from a file
$ oc process my-cache-service -o yaml \
  --param-file=my-cache-service-params.env > my-cache-service-manifest.yaml

$ oc create -f my-cache-service.yaml
$ oc create -f my-cache-service.yaml -n shared-templates
$ oc get templates -n shared-templates
$ oc new-app --template=my-cache-service
```

##### Using Helm

```shell
$ helm show chart chart-reference
$ helm show values chart-reference

$ helm install release-name chart-reference  --dry-run \
  --values values.yaml

$ helm list
$ helm history release_name
$ helm rollback release_name revision

# helm Repositories
$ helm repo add \
  openshift-helm-charts https://charts.openshift.io/
$ helm search repo
```

### Chapter 3.  Authentication and Authorization
#### Objectives
* Configure the HTPasswd identity provider for OpenShift authentication.
* Define role-based access controls and apply permissions to users.

Identity Provider

```shell
$ oc whoami

# Cluster Admin without logging into Openshift
$ export KUBECONFIG=/home/user/auth/kubeconfig

# Update Oauth CR
$ oc get oauth cluster -o yaml > oauth.yaml
# Update locally and run
$ oc replace -f oauth.yaml

# Create the htpasswd file.
# -c = create new file. remove when adding to existing file
# -B = use bcrypt. no need to pass this after initial line
$ htpasswd -c -B -b /tmp/htpasswd student redhat123

$ oc create secret generic htpasswd-secret \
  --from-file htpasswd=/tmp/htpasswd -n openshift-config

# Yo update, extract the file and update locally, then set the data with oc
# Ex. To delete, you must remove the password from the htpasswd secret, remove the user from the local htpasswd file, and then update the secret.
$ oc extract secret/htpasswd-secret -n openshift-config \
  --to /tmp/ --confirm
/tmp/htpasswd

$ htpasswd -D /tmp/htpasswd manager

$ oc set data secret/htpasswd-secret \
  --from-file htpasswd=/tmp/htpasswd -n openshift-config

$ oc delete user manager

# OAuth Operator will restart the pods
$ watch $ oc get pods -n openshift-authentication

# Assigning Administrative Privileges
$ oc adm policy add-cluster-role-to-user cluster-admin student
```

##### Role-based Access Control (RBAC)

Authorization Process

| RBAC Object  | Description  |
|---|---|
| Rule  | Allowed actions for objects or groups of objects.  |
| Role  | Sets of rules. Users and groups can be associated with multiple roles.  |
| Binding  | Assignment of users or groups to a role.  |

Default Roles

| Default roles                 | Description  |
|------------------|---|
| admin            | Users with this role can manage all project resources, including granting access to other users to access the project.  |
| basic-user       | Users with this role have read access to the project.  |
| cluster-admin    | Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.  |
| cluster-status   | Users with this role can get cluster status information.  |
| edit             | Users with this role can create, change, and delete common application resources on the project, such as services and deployments. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.  |
| self-provisioner | Users with this role can create projects. It is a cluster role, not a project role.  |
| view             | Users with this role can view project resources, but cannot modify project resources.  |

##### Managing RBAC with the CLI
```shell
# To add a cluster role to a user, use the add-cluster-role-to-user subcommand:
$ oc adm policy add-cluster-role-to-user cluster-role username

# For example, to change a regular user to a cluster administrator, use the following command:
$ oc adm policy add-cluster-role-to-user cluster-admin username

# To remove a cluster role from a user, use the remove-cluster-role-from-user subcommand:
$ oc adm policy remove-cluster-role-from-user cluster-role username

# For example, to change a cluster administrator to a regular user, use the following command:
$ oc adm policy remove-cluster-role-from-user cluster-admin username

# Rules are defined by an action and a resource. For example, the create user rule is part of the cluster-admin role.
# You can use the oc adm policy who-can command to determine whether a user can execute an action on a resource. For example:
$ oc adm policy who-can delete user

# Group Mgmt
$ oc adm groups new lead-developers
$ oc adm groups add-user lead-developers user1
```

### Chapter 4.  Network Security
#### Objectives
* Allow and protect network connections to applications inside an OpenShift cluster.
* Restrict network traffic between projects and pods.
* Configure and use automatic service certificates.

```shell
# Create an Edge route
# If no cert and key are passed, OCP uses it's own cluster keys
$ oc create route edge \
    --service api-frontend --hostname api.apps.acme.com \
    --key api.key --cert api.crt

# Get interface name 
$ ip addr | grep 172.25.250.9
inet 172.25.250.9/24 brd 172.25.250.255 scope global noprefixroute *eth0*

$ sudo tcpdump -i eth0 -A -n port 80 | grep "angular"

# Create a Pass through route
# Generate private key
$ openssl genrsa -out training.key 4096

# Generate CSR
$ openssl req -new \
    -key training.key -out training.csr \
    -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/\
    CN=todo-https.apps.ocp4.example.com"

# Generate the signed certificate
$ openssl x509 -req -in training.csr \
    -passin file:passphrase.txt \
    -CA training-CA.pem -CAkey training-CA.key -CAcreateserial \
    -out training.crt -days 1825 -sha256 -extfile training.ext

# Create a tls OpenShift secret 
$ oc create secret tls todo-certs \
    --cert certs/training.crt --key certs/training.key

# Update the manifests to mount the certs as a secret
$ cat todo-app-v2.yaml
apiVersion: apps/v1
kind: Deployment
...output omitted...
        volumeMounts:
        - name: tls-certs
          readOnly: true
          mountPath: /usr/local/etc/ssl/certs
...output omitted...
      volumes:
      - name: tls-certs
        secret:
          secretName: todo-certs
---
apiVersion: v1
kind: Service
...output omitted...
  ports:
  - name: https
    port: 8443
    protocol: TCP
    targetPort: 8443
...output omitted...

$ oc create -f todo-app-v2.yaml

# Review the volumes
$ oc set volumes deployment/todo-https
 todo-https
  secret/todo-certs as tls-certs
    mounted at /usr/local/etc/ssl/certs
    
$ Create the route
$ oc create route passthrough todo-https \
    --service todo-https --port 8443 \
    --hostname todo-https.apps.ocp4.example.com
```

##### Network Policies
```shell

# The following has two layers of gatekeeping:
#   1. a label for the pods under podSelector  
#   2. a label on the namespace under namespaceSelector
$ cat np.yaml
...output omitted...
spec:                                       # Three main areas ...
  podSelector:                              # 1. allows traffic into the hello pod
    matchLabels:
      deployment: hello
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            network: different-namespace    # 2. from this namespace
        podSelector:
          matchLabels:
            deployment: sample-app          # 3. and this pod
      ports:
      - port: 8080
        protocol: TCP
```

##### Protect Internal Traffic with TLS
```shell
# generate a certificate and key pair, apply the 
# 'service.beta.openshift.io/serving-cert-secret-name=your-secret'
# annotation to a service
$ oc annotate service hello \ 1
    service.beta.openshift.io/serving-cert-secret-name=hello-secret 2
service/hello annotated

# mount the secret in the application deployment
spec:
  template:
    spec:
      containers:
        - name: hello
          volumeMounts:
            - name: hello-volume 
              mountPath: /etc/pki/nginx/ 
      volumes:
        - name: hello-volume 
          secret:
            defaultMode: 420 
            secretName: hello-secret 
            items:
              - key: tls.crt 
                path: server.crt 
              - key: tls.key 
                path: private/server.key 

# Client Service Application Configuration
# application needs the CA bundle that signed that certificate
# Can be applied to configuration maps, API services, custom resource definitions (CRD),
#   mutating webhooks, and validating webhooks.
$ oc create configmap ca-bundle

$ oc annotate configmap ca-bundle \
    service.beta.openshift.io/inject-cabundle=true
    
# Add to client pod
apiVersion: v1
kind: Pod
metadata:
  name: client
spec:
  containers:
    - name: client
      image: registry.ocp4.example.com:8443/redhattraining/hello-world-nginx
      resources: {}
      volumeMounts:
        - mountPath: /etc/pki/ca-trust/extracted/pem
          name: trusted-ca
  volumes:
    - configMap:
        defaultMode: 420
        name: ca-bundle
        items:
          - key: service-ca.crt
            path: tls-ca-bundle.pem
      name: trusted-ca
```

### Chapter 5.  Expose non-HTTP/SNI Applications
#### Objectives
* Expose applications to external access by using load balancer services.
* Expose applications to external access by using a secondary network.

##### Load Balancer Services
Ensure the LB configuration has adequate IP address range
```shell
$ oc expose deployment/virtual-rtsp-1 \
  --type=LoadBalancer --target-port=8554
```

##### Multus Secondary Networks
```shell
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: custom
spec:
  config: |-
    {
      "cniVersion": "0.3.1",
      "name": "custom",
      "type": "host-device",
      "device": "ens4",
      "ipam": {
        "type": "static",
        "addresses": [
          {"address": "192.168.51.10/24"}
        ]
      }
    }
    
$ oc create -f \
  network-attachment-definition.yaml
  
# Add to deployment via annotation
...output omitted...
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        k8s.v1.cni.cncf.io/networks: custom
    spec:
      containers:
...output omitted...

$ oc apply -f nginx.yaml

$ oc get pod nginx-6f45d9f89-wp2gg \
  -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/networks-status}'
```

### Chapter 6.  Enable Developer Self-Service
#### Objectives
* Configure compute resource quotas and Kubernetes resource count quotas per project and cluster-wide.
* Configure default and maximum compute resource requirements for pods per project.
* Configure default quotas, limit ranges, role bindings, and other restrictions for new projects, and the allowed users to self-provision new projects.

##### Project and Cluster Quotas
```shell
$ oc create resourcequota example --hard=count/pods=1
resourcequota/example created

$ oc get resourcequota example -o yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: example
spec:
  hard:
    count/pods: "1"
    
$ oc get quota
NAME      AGE     REQUEST           LIMIT
example   9m54s   count/pods: 1/1

$ oc scale deployment example --replicas=8

#Troubleshoot node and quotas
$ oc adm top node
$ oc describe node/master01
```

##### Per-Project Resource Constraints: Limit Ranges
```shell
$ oc get limitrange example -o yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: example
  namespace: example
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 250m
      memory: 256Mi
    max:
      cpu: "1"
      memory: 1Gi
    min:
      cpu: 125m
      memory: 128Mi
    type: Container
```

##### The Project Template and the Self-Provisioner Role
Set up project defaults

Make changes to:
* Roles and Rolebindings
* Resource quotas and ranges
* Network Policies

###### Create template
```shell
$ oc adm create-bootstrap-project-template \
  -o yaml >template.yaml

# Create quota
$ oc create quota memory \
  --hard=requests.memory=2Gi,limits.memory=4Gi \
  -n template-test
  
# Create limitrange (from file only)
$ oc create \
  -f ~/DO280/labs/selfservice-review/limitrange.yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: memory
  namespace: template-test
spec:
  limits:
  - min:
      memory: 128Mi
    defaultRequest:
      memory: 256Mi
    default:
      memory: 512Mi
    max:
      memory: 1Gi
    type: Container

# Append linitrange to template.yaml
oc get limitrange,quota -n template-test \
  -o yaml >>template.yaml
  
# Tidy up template.yaml

# Create and configure the project template
$ oc create -f template.yaml -n openshift-config

# change the global cluster project configuration
$ oc edit projects.config.openshift.io cluster

apiVersion: config.openshift.io/v1
kind: Project
metadata:
...output omitted...
  name: cluster
...output omitted...
spec:
  projectRequestTemplate:
    name: project-request     # <------ Add template name here

# Save and api-server pods will restart
$ watch oc get pod -n openshift-apiserver
```

### Chapter 7.  Manage Kubernetes Operators
#### Objectives
* Explain the operator pattern and different approaches for installing and updating Kubernetes operators.
* Install and update operators by using the web console and the Operator Lifecycle Manager.
* Install and update operators by using the Operator Lifecycle Manager APIs.

```shell
# Create ns
oc create -f ./namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
    pod-security.kubernetes.io/enforce: privileged
  name: openshift-file-integrity
  
# Create an operator group in the operator namespace
$ oc create -f ./operator-group.yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: file-integrity-operator
  namespace: openshift-file-integrity
spec:
  targetNamespaces:
  - openshift-file-integrity
  
oc create -f ./subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: file-integrity-operator
  namespace: openshift-file-integrity
spec:
  channel: "stable"
  installPlanApproval: Manual
  name: file-integrity-operator
  source: do280-catalog-redhat
  sourceNamespace: openshift-marketplace
  
# Approve the installPlan
$ oc patch installplan install-pmh78 --type merge -p \
    '{"spec":{"approved":true}}' -n openshift-file-integrity
installplan.operators.coreos.com/install-pmh78 patched
```

### Chapter 8.  Application Security
#### Objectives
* Create service accounts and apply permissions, and manage security context constraints.
* Run an application that requires access to the Kubernetes API of the application's cluster.
* Automate regular cluster and application management tasks by using Kubernetes cron jobs.

##### Control Application Permissions with Security Context Constraints
```shell
# Create SA
$ oc create sa gitlab-sa

# Assign the anyuid SCC to the sa
$ oc adm policy add-scc-to-user anyuid -z gitlab-sa

# Assign the gitlab-sa service account to the gitlab deployment
$ oc set serviceaccount deployment/gitlab gitlab-sa
```

##### Allow Application Access to Kubernetes APIs
```shell
$ oc create sa configmap-reloader

# Add sa to deployment
...output omitted...
    spec:
      serviceAccountName: configmap-reloader
      containers:
...output omitted...

# Assign the edit cluster role to the configmap-reloader service account in the appsec-api project
$ oc adm policy add-role-to-user edit \
   system:serviceaccount:configmap-reloader:configmap-reloader \
   --rolebinding-name=reloader-edit \
   -n appsec-api
```

##### Cluster and Node Maintenance with Kubernetes Cron Jobs
```shell
$ oc debug node/master01 -- \
  chroot /host crictl images | egrep '^IMAGE|httpd|nginx'

$ oc debug node/master01 -- \
  chroot /host crictl rmi --prune

$ oc delete deployment nginx-ubi{7,8,9} \
  -n prune-apps

# Automate this
# Create configmap with prune cmd
apiVersion: v1
kind: ConfigMap
metadata:
  name: maintenance
  labels:
    ge: appsec-prune
    app: crictl
data:
  maintenance.sh: |
    #!/bin/bash -eu
    NODES=$(oc get nodes -o=name)
    for NODE in ${NODES}
    do
      echo ${NODE}
      oc debug ${NODE} -- \
        chroot /host \
          /bin/bash -euxc 'crictl images ; crictl rmi --prune'
    done

$ oc apply -f configmap-prune.yaml

# Create configmap
apiVersion: batch/v1
kind: CronJob
metadata:
  name: image-pruner
  labels:
    ge: appsec-prune
    app: crictl
spec:
  schedule: '*/4 * * * *'
  jobTemplate:
    spec:
      template:
        spec:
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          containers:
          - name: crictl
            image: registry.ocp4.example.com:8443/openshift/origin-cli:4.12  1
            resources: {}
            command:
            - /opt/maintenance.sh
            volumeMounts:
            - name: scripts
              mountPath: /opt
          volumes:
          - name: scripts
            configMap:
              name: maintenance
              defaultMode: 0555
              
$ oc apply -f configmap-prune.yaml              

# Set the appropriate permissions to run the image pruner cron job
$ oc adm policy add-scc-to-user -z default privileged

$ oc adm policy add-cluster-role-to-user \
  cluster-admin -z default
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "default"
```



### Chapter 9.  OpenShift Updates
#### Objectives
* Describe the cluster update process.
* Identify applications that use deprecated Kubernetes APIs.
* Update OLM-managed operators by using the web console and CLI.
