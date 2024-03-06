# EX180
## Introduction
This course prepares OpenShift Cluster Administrators to perform day-to-day management of Kubernetes workloads and collaborate with Developers, DevOps Engineers, System Administrators, and SREs to ensure the availability of application workloads. DO180 focuses on managing typical end-user applications. These applications are accessible from a web or mobile UI and represent the majority of cloud native and containerized workloads. Management of applications also include deployment of their dependencies such as databases, messaging, and authentication systems. This course is based on Red Hat® OpenShift® Container Platform 4.14.

### Course Objectives
* Managing OpenShift clusters from the command-line interface and from the web console.
* Deploying applications on OpenShift from container images, templates, and Kubernetes manifests.
* Troubleshooting network connectivity between applications inside and outside an OpenShift cluster.
* Connecting Kubernetes workloads to storage for application data.
* Configuring Kubernetes workloads for high availability and reliability.
* Managing updates to container images, settings, and Kubernetes manifests of an application.

### Chapter 2.  Kubernetes and OpenShift Command-line Interfaces and APIs
#### Objectives
* Access an OpenShift cluster by using the Kubernetes and OpenShift command-line interfaces.
* Query, format, and filter attributes of Kubernetes resources.
* Query the health of essential cluster services and components.

```bash
kubectl explain pod

oc describe mysql-openshift-1-glgrp

oc api-resources

oc get pods -o yaml

oc get pods -o yaml | yq r - 'items[0].status.podIP'

oc get pods -o json

oc get pods \
  -o custom-columns=PodName:".metadata.name",\
     ContainerName:"spec.containers[].name",\
     Phase:"status.phase",\
     IP:"status.podIP",\
     Ports:"spec.containers[].ports[].containerPort"

oc get pods  \
  -o jsonpath='{range .items[]}{"Pod Name: "}{.metadata.name}{"IP: "}{.status.podIP}{"Ports: "}{.spec.containers[].ports[].containerPort}{"\n"}{end}'

oc get clusteroperators

oc get clusteroperators dns -o yaml

# Examine cluster Metrics
oc adm top pods -A --sum
NAMESPACE                 NAME                      CPU(cores)   MEMORY(bytes)
metallb-system            controller-...-ddr8v      0m           57Mi
metallb-system            metallb-...-n2zsv         0m           48Mi
...output omitted...
openshift-storage         topolvm-node-9spzf        0m           68Mi
openshift-storage         vg-manager-z8g5k          0m           23Mi
                                                   ------       --------
                                                    428m         10933Mi

oc adm top pods etcd-master01 -n openshift-etcd --containers
POD             NAME           CPU(cores)   MEMORY(bytes)
etcd-master01   POD            0m           0Mi
etcd-master01   etcd           71m          933Mi
etcd-master01   etcd-metrics   6m           32Mi
etcd-master01   etcd-readyz    4m           66Mi
etcd-master01   etcdctl        0m           0Mi

# Query Cluster Events and Alerts
oc get events -n openshift-kube-controller-manager

# Kubernetes Alerts
oc get all -n openshift-monitoring --show-kind

# Check Node Status
oc cluster-info

oc get nodes

oc get node master01 -o jsonpath=\
*'{"Allocatable:\n"}{.status.allocatable}{"\n\n"}{"Capacity:\n"}{.status.capacity}{"\n"}'
Allocatable:
{"cpu":"7500m","ephemeral-storage":"114396791822","hugepages-1Gi":"0",
"hugepages-2Mi":"0","memory":"19380692Ki","pods":"250"}

Capacity:
{"cpu":"8","ephemeral-storage":"125293548Ki","hugepages-1Gi":"0",
"hugepages-2Mi":"0","memory":"20531668Ki","pods":"250"}

oc get node master01 -o json | jq '.status.conditions'
[
  {
    "lastHeartbeatTime": "2023-03-22T16:34:57Z",
    "lastTransitionTime": "2023-02-23T20:35:15Z",
    "message": "kubelet has sufficient memory available",
    "reason": "KubeletHasSufficientMemory",
    "status": "False",
    "type": "MemoryPressure" 1
  }
]

# Deeper Insight to a given Node
oc adm node-logs <FILTER>

--role master	  Use the --role option to filter the output to nodes with a specified role.
-u kubelet	      The -u option filters the output to a specified unit.
--path=cron	      The --path option filters the output to a specific process under the /var/logs directory.
--tail 1	      Use --tail x to limit output to the last x log entries.

oc debug node/node-name

# Within the debug session, change to the /host root directory so that you can run binaries in the host's executable path.
sh-4.4# chroot /host
sh-4.4# systemctl status kubelet

# Pod Status
oc logs pod-name -c container-name

# Collect Information for Support Requests
oc adm must-gather --dest-dir /home/student/must-gather
tar cvaf mustgather.tar must-gather/
oc adm inspect clusteroperator/openshift-apiserver clusteroperator/kube-apiserver
oc adm inspect clusteroperator/openshift-apiserver --since 10m



```