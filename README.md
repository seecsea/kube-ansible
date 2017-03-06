# My Kubernetes and Ceph development environment
A vagrant development environment for Kubernetes and Ceph

If you want use this vagrant files, you must install the following of plugins:
```sh
$ vagrant plugin install vagrant-libvirt
```

TODO:
- [x] Vagrant scripts.
- [x] Kubernetes cluster setup(v1.5.0+).
- [x] Ceph cluster on Kubernetes(v11.2.0+).
- [ ] Kubernetes Ceph RBD/FS volume.
- [ ] Support vagrant-openstack.

## Quick Start
Following the below steps to create Kubernetes setup on `CentOS 7.x` and `Ubuntu Server 16.x` .

The getting started guide will use Vagrant with VirtualBox. It can deploy your Kubernetes cluster with a single command:
```sh
$ ./setup-vagrant.sh
```

### Requirement
* Deploy node need install Ansible.
* All master/node should have password-less access from Deploy node.

### VM and BareMetal Setup
Add the system information gathered above into a file called inventory.

For inventory example:
```
[etcd]
172.16.35.13

[masters]
172.16.35.13

[sslhost]
172.16.35.13

[nodes]
172.16.35.10
172.16.35.11
172.16.35.12
```

Set the variables in `group_vars/all.yml` to reflect you need options.

When variables are already set, just execute `cluster.yml` to deploy cluster:
```sh
$ ansible-playbook -i inventory cluster.yml
```

When cluster is fully operation and running, just execute `addons.yml` to create addons:
```sh
$ ansible-playbook -i inventory addons.yml
```

If you want to deploying a Ceph cluster on to a Kubernetes cluster, just execute `ceph-cluster.yml`:
```sh
$ ansible-playbook -i inventory ceph-cluster.yml
```

When ceph cluster is fully running, you must label your storage nodes in order to run Ceph pods on them:
```sh
$ kubectl label node <nodename> node-type=storage
```

## Verify cluster
If all step completed, you can run following the below command:
```sh
$ kubectl get po,svc --namespace=kube-system

NAME                                 READY     STATUS    RESTARTS   AGE       IP             NODE
po/haproxy-master1                   1/1       Running   0          2h        172.16.35.13   master1
po/kube-apiserver-master1            1/1       Running   0          2h        172.16.35.13   master1
po/kube-controller-manager-master1   1/1       Running   1          2h        172.16.35.13   master1
po/kube-dns-v20-sp2xj                3/3       Running   0          2h        172.20.3.2     node3
po/kube-proxy-amd64-4g4kn            1/1       Running   0          2h        172.16.35.12   node3
po/kube-proxy-amd64-cqbnk            1/1       Running   0          2h        172.16.35.11   node2
po/kube-proxy-amd64-d7l1p            1/1       Running   0          2h        172.16.35.10   node1
po/kube-proxy-amd64-f2wqq            1/1       Running   0          2h        172.16.35.13   master1
po/kube-scheduler-master1            1/1       Running   2          2h        172.16.35.13   master1

NAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE       SELECTOR
svc/kube-dns   192.160.0.10   <none>        53/UDP,53/TCP   2h        k8s-app=kube-dns
```

Check ceph cluster is running:
```sh
$ kubectl get po,svc --namespace=ceph

NAME                                 READY     STATUS    RESTARTS   AGE       IP            NODE
po/ceph-mds-2743106415-gccj5         1/1       Running   0          1h        172.20.67.4   node1
po/ceph-mon-246094207-6r9g6          1/1       Running   0          1h        172.20.67.2   node1
po/ceph-mon-246094207-hx0md          1/1       Running   1          1h        172.20.77.3   node2
po/ceph-mon-246094207-pv3b0          1/1       Running   0          1h        172.20.3.3    node3
po/ceph-mon-check-1896585268-4m9sw   1/1       Running   0          1h        172.20.77.2   node2
po/ceph-osd-5m9hw                    1/1       Running   0          1h        172.20.3.4    node3
po/ceph-osd-qn5qt                    1/1       Running   0          1h        172.20.77.4   node2
po/ceph-osd-r7251                    1/1       Running   0          1h        172.20.67.3   node1

NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE       SELECTOR
svc/ceph-mon   None         <none>        6789/TCP   1h        app=ceph,daemon=mon
```

Get ceph status using kubectl exec:
```sh
$ kubectl --namespace=ceph exec -ti ceph-mon-246094207-6r9g6 -- ceph -s

cluster bafca3e9-b361-464c-b8fa-04bf60b3189f
 health HEALTH_OK
 monmap e2: 1 mons at {ceph-mon-246094207-6r9g6=172.20.67.2:6789/0}
        election epoch 4, quorum 0 ceph-mon-246094207-6r9g6
  fsmap e5: 1/1/1 up {0=mds-ceph-mds-2743106415-gccj5=up:active}
    mgr no daemons active
 osdmap e17: 3 osds: 3 up, 3 in
        flags sortbitwise,require_jewel_osds,require_kraken_osds
  pgmap v1813: 80 pgs, 3 pools, 2148 bytes data, 20 objects
        32751 MB used, 83338 MB / 113 GB avail
              80 active+clean
```

## Nginx application example
Run a simple nginx application:
```sh
$ kubectl create -f examples/nginx/
$ kubectl get svc,po -o wide
```
