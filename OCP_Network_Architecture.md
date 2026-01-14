### The front door to the cluster.

[root@mfau-thinkpadx1carbon5th ~]# oc get ingresscontroller -n openshift-ingress-operator
NAME      AGE
default   4h53m

### oc get network cluster -o yaml 

[root@mfau-thinkpadx1carbon5th ~]# oc get network cluster -o yaml
apiVersion: config.openshift.io/v1
kind: Network
metadata:
  creationTimestamp: "2025-12-23T15:08:06Z"
  generation: 3
  name: cluster
  resourceVersion: "29385"
  uid: 36593ca3-6a00-42ff-8eca-26c48ac09131
spec:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  externalIP:
    policy: {}
  networkDiagnostics:
    mode: ""
    sourcePlacement: {}
    targetPlacement: {}
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
status:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  clusterNetworkMTU: 1400
  conditions:
  - lastTransitionTime: "2025-12-23T15:48:32Z"
    message: ""
    observedGeneration: 0
    reason: AsExpected
    status: "True"
    type: NetworkDiagnosticsAvailable
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16

### Masquerade subnet for SNAT (outbound traffic from pods)

+-------------------+          +---------------------+          +---------------------+
|   Pod (10.128.x.x)|          |      Node IP        |          |  External Network   |
|-------------------|          |---------------------|          |---------------------|
| Pod wants to talk  | ----->   | Masquerade/Subnet:  | ----->   | Destination sees    |
| to external system|          | 169.254.0.0/17      |          | Node IP as source   |
+-------------------+          +---------------------+          +---------------------+

### The br-ex connects the cluster network to the outside world.

Pod
 └── veth
     └── ovn-k8s overlay
         └── node gateway router
             └── br-ex
                 └── physical NIC
                     └── external network

### Mental model.

[ Pod ]
   |
   veth (random name)
   |
[ ovs-system ]
   |
[ br-int ] ---- geneve ---- [ br-int ] (other nodes)
   |
[ ovn-k8s-mp0 ]  (pod subnet gateway)
   |
[ br-ex ]
   |
[ enp1s0 ]
   |
[ External network ]
..................................................
### Where does the ovn-k8s-mp0 fit in the network diagram?

- Good question — ovn-k8s-mp0 is the node’s management port, and it sits between the node gateway router and the host networking stack.
- Here’s how it fits into your diagram, with one small but important refinement.
- Where ovn-k8s-mp0 lives

Your flow becomes:
```
Pod
 └── veth
     └── OVN logical switch (ovn-k8s overlay)
         └── Node Gateway Router (logical)
             └── ovn-k8s-mp0   ← ★ management port ★
                 └── br-ex
                     └── physical NIC
                         └── external network
```
### What ovn-k8s-mp0 actually is

- ovn-k8s-mp0 is a Linux interface on the node
- It represents the management port of the node’s OVN gateway router
- It is how OVN hands packets to the host so Linux can:
  - Apply routing
  - Apply NAT (SNAT/DNAT)
  - Send traffic out via br-ex and the physical NIC

### Why it matters

- Pod traffic does not go directly from OVN to br-ex
- It must pass through:
  - the logical gateway router (OVN)
  - then the management port (ovn-k8s-mp0)
  - then into Linux networking (br-ex)

Think of ovn-k8s-mp0 as:

“The plug where OVN connects into the node’s real network stack.”

One-sentence summary

ovn-k8s-mp0 sits between the OVN node gateway router and br-ex, acting as the management interface that connects OVN’s logical networking to the host’s physical network.