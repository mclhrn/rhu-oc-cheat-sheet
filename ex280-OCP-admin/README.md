[//]: # (TODO Next exam)


# Networking
## Exposing Routes
#### Securing Services with a given hostname
```
oc expose service <ROUTE_NAME> --hostname <NAME>.apps.ocp4.example.com
```
#### Debug deployment
```
oc debug -t deployment/<NAME>
```

### Securing Applications
#### Securing Applications with Edge Routes
```
oc create route edge \
    --service api-frontend \
    --hostname api.apps.acme.com \
    --key api.key \
    --cert api.crt
```
#### Securing Applications with Passthrough Routes
```
oc create route passthrough route-passthrough-secured \
    --service=api-frontend \
    --port=8080
```

<hr>

## Network Policies
### Print Project Network Details

```bash
#!/usr/bin/bash
oc get pods -o custom-columns="POD NAME:.metadata.name,IP ADDRESS:.status.podIP" -n <PROJECT_NAME>
oc get svc -o custom-columns="SERVICE NAME:.metadata.name,CLUSTER-IP:.spec.clusterIP" -n <PROJECT_NAME>
oc get route -o custom-columns="ROUTE NAME:.metadata.name,HOSTNAME:.spec.host,PORT:.spec.port.targetPort" -n <PROJECT_NAME>
```

### Confirm Access
#### Pod to Pod - Same Namespce
```
oc rsh <POD_NAME> curl <TARGET_POD_IP>:8080 | grep <GREP_TERM>
```

#### Pod to Service - Same Namespce
```
oc rsh <POD_NAME> curl <SERVICE_IP>:8080 | <GREP_TERM>
```

### Network Policy
#### Deny all pods in a namespace
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all
spec:
  podSelector: {}
```

#### Allow traffic _to_ specific pod from a pod in different ns
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      deployment: hello # pod label
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: network-test # namespace label
        podSelector:
          matchLabels:
            deployment: sample-app # pod label
      ports:
      - port: 8080
        protocol: TCP
```

#### Allow traffic _to_ specific pod from exposed route
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-from-openshift-ingress
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: network.openshift.io/policy-group: ingress
```
<hr>


# Scheduling
## Controllong Pod Scheduling

#### Manually scale an application
```
oc scale --replicas 4 deployment/<NAME>
```

#### Assigning label to nodes
```
oc get nodes -L env # list the nodes and labels with the key 'env'
oc label node master01 env=dev # user defined
oc label node master02 env=prod # user defined
```

#### Removing label from nodes
```
oc label node -l env env-
```

#### Assigning node selector to deployment
```
oc edit deployment/<NAME>
```
```
...output omitted...
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:
        env: dev
      restartPolicy: Always
...output omitted...

```
Or use `oc patch` to accomplish the same thing
```
oc patch deployment/<NAME> --patch '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'
```

## Limiting Resource Usage by Application
#### Set Resource Limits
```
oc set resources deployment hello-world-nginx --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
```

#### Display Requests and Limits for Worker Nodes
```
oc adm top nodes|pods|images|imagestreams -l node-role.kubernetes.io/worker
```

#### Create a resource quota
```
oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi
```

#### Other resourcequota commands
```
oc get resourcequota
oc describe quota
oc delete resourcequota <NAME>
```

### Applying Limit Ranges
#### Create a limit range resource
```
oc create --save-config -f dev-limits.yml
```
dev-limits.yml:
```
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "dev-limits"
spec:
  limits:
    - type: "Pod"
      max: 1
        cpu: "500m"
        memory: "750Mi"
      min: 2
        cpu: "10m"
        memory: "5Mi"
    - type: "Container"
      max: 3
        cpu: "500m"
        memory: "750Mi"
      min: 4
        cpu: "10m"
        memory: "5Mi"
      default: 5
        cpu: "100m"
        memory: "100Mi"
      defaultRequest: 6
        cpu: "20m"
        memory: "20Mi"
    - type: openshift.io/Image 7
      max:
        storage: 1Gi
    - type: openshift.io/ImageStream 8
      max:
        openshift.io/image-tags: 10
        openshift.io/images: 20
    - type: "PersistentVolumeClaim" 9
      min:
        storage: "1Gi"
      max:
        storage: "50Gi"
```

#### Other limitrange commands
```
oc describe limitrange dev-limits
oc delete limitrange dev-limits
```

### Applying Quotas to Multiple Projects

#### ClusterResourceQuota at user level
```
oc create clusterquota user-qa \
  --project-annotation-selector openshift.io/requester=qa \
  --hard pods=12,secrets=20
```

#### ClusterResourceQuota at user level
```
oc create clusterquota env-qa \
  --project-label-selector environment=qa \
  --hard pods=10,services=5
```

#### Other clusterquota commands
```
oc delete clusterquota QUOTA
```

### Customizing the Default Project Template
Redirect the template to a file and customise
```
oc adm create-bootstrap-project-template \
  -o yaml > /tmp/project-template.yaml
```
project-template.yaml
```
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: ${PROJECT_NAME}-quota
  spec:
    hard:
      cpu: "3"
      memory: 10Gi
      pods: "10"
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
```
Create the template with changes
```
oc create -f /tmp/project-template.yaml -n openshift-config
```
