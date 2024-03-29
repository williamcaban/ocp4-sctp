# Enabling SCTP on OpenShift 4.2.x

```
#########################################################################
 WARNING: THIS PROCEDURE CANNOT BE UNDONE AND PREVENTS CLUSTER UPGRADES.
** DO NOT USE THIS PROCEDURE WITH OCP 4.3.x SCTP FEATUREGATE AVAILABLE **
#########################################################################
```

- Create a FeatureGate resource to enable the SCTP Kubernetes Alpha feature.
    ```
    oc create -f 00-featuregate.yaml
    ```

- Deploy MachineConfig to whitelist SCTP Kernel Module
    ```
    oc create -f 02-allow-sctp.yaml
    ```

- Wait for the Operator to complete the configuration update

## Testing an SCTP Demo App

- Create project for demo app 
    ```
    oc new-project sctp-demo
    ```

- Create a Service Account with cluster-admin privileges (required for now for SCTP sockets)
    NOTE: This is NOT necesary in OCP 4.3.x versions
    ```
    oc create sa sctp-sa
    oc adm policy add-role-to-user cluster-admin -z sctp-sa
    ```

- Deploy a simple STCP demo app:
    ```
    oc create -f 05-sctp-app-deployment.yaml
    ```
- Validate the pod is running
    ```
    # oc get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    sctpserver-59bf4d8b87-nvn85   1/1     Running   0          3m40s
    ```
- Login into the Pod and validate SCTP is supported. If so, `sctp_test` can be run
    ```
    # Login into the Pod
    [root@bastion ocp4-sctp]# oc exec -ti sctpserver-59bf4d8b87-nvn85 bash

    # Validating SCTP is supported
    [root@sctpserver-59bf4d8b87-nvn85 /]# checksctp
    SCTP supported

    # Running the sctp_test
    [root@sctpserver-59bf4d8b87-nvn85 /]# sctp_test -H localhost -P 30100 -l
    local:addr=::, port=rwp, family=10
    seed = 1572022216

    Starting tests...
            socket(SOCK_SEQPACKET, IPPROTO_SCTP)  ->  sk=3
            bind(sk=3, [a:::,p:rwp])  --  attempt 1/10
            listen(sk=3,backlog=100)
    Server: Receiving packets.
            recvmsg(sk=3)
    ```
- In another shell login to the same Pod and validate is listening with SCTP protocol
    ```
    # netstat --sctp -atplun
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    sctp                :::30100                                        LISTEN      29/sctp_test
    ```

- Deploy an STCP Service resrouce:
    ```
    oc create -f 05-sctp-app-svc.yaml
    ```
- Validate the service shows SCTP protocol for the ports:
    ```
    # oc get svc sctpserver
    NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)            AGE
    sctpserver   NodePort   172.30.134.99   <none>        30100:30100/SCTP   3s
    ```

## Troubleshooting

- To troubleshoot or validate SELinux issues
    ```
    journalctl -b _TRANSPORT=kernel --grep=avc:
    ```

# Acknowledgements

Thanks to Ricardo Noriega who helped with some of the troubleshootings. Take a look to his work at the [OpenAir Interface project](https://github.com/OPENAIRINTERFACE/openair-k8s/tree/master/manifests)
