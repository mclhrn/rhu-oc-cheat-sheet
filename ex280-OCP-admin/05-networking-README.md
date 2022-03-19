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
