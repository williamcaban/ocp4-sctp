# Enabling SCTP on OpenShift 4.2.x

```
#########################################################################
 WARNING: THIS PROCEDURE CANNOT BE UNDONE AND PREVENTS CLUSTER UPGRADES.
#########################################################################
```

- Create a FeatureGate resource to enable the SCTP Kubernetes Alpha feature.

  ```
  oc create -f 00-featuregate.yaml
  ```

- Create project for SCTP feature
    ```
    oc new-project ocp-sctp
    ```

- Create a Service Account with cluster-admin privileges (required for now for SCTP sockets)
  ```
  oc create sa sctp-sa
  oc adm policy add-scc-to-user cluster-admin -z sctp-sa
  ```

- Deploy ConfigMap and DaemonSet to load SCTP Kernel Module into Nodes
    ```
    oc create -f 02-sctp-cm.yaml
    oc create -f 02-sctp-ds.yaml
    ```
- 

## Testing an SCTP Demo App

- Create project for demo app 
    ```
    oc new-project sctp-demo
    ```

-  Use the `system:serviceaccount:ocp-sctp:sctp-sa` created when enabling the feature, to deploy the workload. To deploy a simple STCP demo app and a Service execute:
    ```
    oc create -f 05-sctp-app-deployment.yaml
    oc create -f 05-sctp-app-svc.yaml
    ```

- Validate the service shows SCTP protocol for the ports:
    ```
    # oc get svc sctpserver
    NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)            AGE
    sctpserver   NodePort   172.30.134.99   <none>        30100:30100/SCTP   3s
    ```