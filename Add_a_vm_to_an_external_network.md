### 1. Create a linux bridge that attaches to an external network using an nncp.
### 2. Create a nad in a specific vm project.
### 3. Create a vm to use the linux bridge.

### Add a label to the worker nodes to match a nncp.

[student@workstation ~]$ oc label node worker01 orgnet=true
[student@workstation ~]$ oc label node worker02 orgnet=true
node/worker02 labeled

### Create the nncp.
```
vim nncp_linuxbr_br0.yaml

apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: br0-ens4-policy
spec:
  desiredState:
    interfaces:
    - bridge:
        port:
        - name: ens4
      ipv4:
        dhcp: true
        enabled: true
      name: br0
      state: up
      type: linux-bridge
  nodeSelector:
    orgnet: "true"
```

```
oc apply -f nncp_linuxbr_br0.yaml
```
### Create a nad.
```
vim nad_linuxbr_br0.yaml

apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: ex-net
  namespace: review-cr1
spec:
  config: '{"name":"ex-net","type":"bridge","cniVersion":"0.3.1","bridge":"br0","macspoofchk":true,"ipam":{},"preserveDefaultVlan":false}'
```
```
oc apply -f nad_linuxbr_br0.yaml
```
### Before I applied the nap to the vm. It just had a pod network ip.
```
[student@workstation ~]$ oc get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
virt-launcher-web1-747l2   1/1     Running   0          13m   10.8.0.42   master03   <none>           1/1
```
### This shows the cluster network config....very cool.
```
[student@workstation ~]$ oc get network cluster -o yaml
apiVersion: config.openshift.io/v1
kind: Network
metadata:
  creationTimestamp: "2025-04-10T13:16:40Z"
  generation: 13
  name: cluster
  resourceVersion: "395267"
  uid: f4fc6176-66ea-49cc-a826-7be346568c36
spec:
  clusterNetwork:
  - cidr: 10.8.0.0/14
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
  - cidr: 10.8.0.0/14
    hostPrefix: 23
  clusterNetworkMTU: 1400
  conditions:
  - lastTransitionTime: "2025-12-26T03:18:56Z"
    message: ""
    reason: AsExpected
    status: "True"
    type: NetworkDiagnosticsAvailable
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
```
### Create the vm.
```
vim vm_linuxbr_br0.yaml

apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
    kubevirt.io/dynamic-credentials-support: "true"
    vm.kubevirt.io/template: rhel9-server-small
    vm.kubevirt.io/template.namespace: openshift
    vm.kubevirt.io/template.revision: "1"
    vm.kubevirt.io/template.version: v0.29.2
  name: web1
  namespace: review-cr1
spec:
  dataVolumeTemplates:
  - apiVersion: cdi.kubevirt.io/v1beta1
    kind: DataVolume
      name: web1
    spec:
      sourceRef:
        kind: DataSource
        name: rhel9
        namespace: openshift-virtualization-os-images
      storage:
        resources:
          requests:
            storage: 30Gi
  running: true
  template:
    metadata:
      annotations:
        vm.kubevirt.io/flavor: small
        vm.kubevirt.io/os: rhel9
        vm.kubevirt.io/workload: server
      creationTimestamp: null
      labels:
        kubevirt.io/domain: web1
        kubevirt.io/size: small
        network.kubevirt.io/headlessService: headless
    spec:
      accessCredentials:
      - sshPublicKey:
          propagationMethod:
            noCloud: {}
          source:
            secret:
              secretName: lab-rsa
      architecture: amd64
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
          - disk:
              bus: virtio
            name: rootdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
          interfaces:
          - macAddress: 02:b0:7d:00:00:06
            masquerade: {}
            model: virtio
            name: default
          - bridge: {}
            macAddress: 02:b0:7d:00:00:07
            model: virtio
            name: nic-0
          rng: {}
        features:
          acpi: {}
          smm:
            enabled: true
        firmware:
          bootloader:
            efi: {}
        machine:
          type: pc-q35-rhel9.4.0
        memory:
          guest: 2Gi
        resources: {}
      networks:
      - name: default
        pod: {}
      - multus:
          networkName: ex-net
        name: nic-0
      nodeSelector:
        orgnet: "true"
      terminationGracePeriodSeconds: 180
      volumes:
      - dataVolume:
          name: web1
        name: rootdisk
      - cloudInitNoCloud:
          userData: |
            #cloud-config
            user: developer
            password: developer
            chpasswd:
              expire: false
        name: cloudinitdisk
```
### After the nad was applied and the vm rebooted, inside the vm it showed the machine network IP.
### I had to log into the utility sever, which was on the same 192.x.x.x network then ssh into the vm.
```
[developer@web1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:b0:7d:00:00:06 brd ff:ff:ff:ff:ff:ff
    altname enp1s0
    inet 10.0.2.2/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 86313513sec preferred_lft 86313513sec
    inet6 fe80::b0:7dff:fe00:6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:b0:7d:00:00:07 brd ff:ff:ff:ff:ff:ff
    altname enp2s0
    inet 192.168.51.102/24 brd 192.168.51.255 scope global dynamic noprefixroute eth1
       valid_lft 380717232sec preferred_lft 380717232sec
    inet6 fe80::c8e0:916:afa0:cc6d/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
### But the pod still shows a pod network IP....interesting.
### Even when I deleted the default network (pod network) from the vm config, it still showed a pod IP, not sure why?
```
[student@workstation ~]$ oc get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP          NODE       NOMINATED NODE   READINESS GATES
virt-launcher-web1-v6sxf   1/1     Running   0          4m8s   10.8.2.31   worker02   <none>           1/1
```

