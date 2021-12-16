# rhu-oc-cheat-sheet
Collection of commands and aliases to help with Openshift Certification Exams

```bash
#!/usr/bin/bash

if oc get project -o jsonpath='{.items[*].metadata.name}' | grep -q <PROJECT_NAME>
then
  echo "==================================================================="
  echo "PROJECT: <PROJECT_NAME>"
  echo
  oc get pods -o custom-columns="POD NAME:.metadata.name,IP ADDRESS:.status.podIP" -n <PROJECT_NAME>
  echo
  oc get svc -o custom-columns="SERVICE NAME:.metadata.name,CLUSTER-IP:.spec.clusterIP" -n <PROJECT_NAME>
  echo
  oc get route -o custom-columns="ROUTE NAME:.metadata.name,HOSTNAME:.spec.host,PORT:.spec.port.targetPort" -n <PROJECT_NAME>
  echo
  echo "==================================================================="
fi

if oc get project -o jsonpath='{.items[*].metadata.name}' | grep -q <PROJECT_NAME>
then
  echo "PROJECT: <PROJECT_NAME>"
  echo
  oc get pods -o custom-columns="POD NAME:.metadata.name" -n <PROJECT_NAME>
  echo
  echo "==================================================================="
fi

```


