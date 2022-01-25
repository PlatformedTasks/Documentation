# Setup a PLAS compatible testbed

This guide is a step by step reference to properly configure a working PLAS compatible testbed:

1. Install `kubectl` version >= v1.21
2. Install an NFS provisioner. We have chosen the NFS Ganesha server as described in the [official guide](https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs). In particular from the [GitHub repository](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner) we have installed only the deployment with its relative RBAC (as the code below shows), while for the storage class will be installed using the Helm Chart of the TESK-API:

```console
$ git clone https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner.git
$ cd nfs-ganesha-server-and-external-provisioner/
$ kubectl create -f deploy/kubernetes/deployment.yaml
serviceaccount/nfs-provisioner created
service "nfs-provisioner" created
deployment "nfs-provisioner" created
$ kubectl create -f deploy/kubernetes/rbac.yaml
clusterrole.rbac.authorization.k8s.io/nfs-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-provisioner created
role.rbac.authorization.k8s.io/leader-locking-nfs-provisioner created
rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-provisioner created
```

3. Install and configure an FTP server
4. Install [Helm](https://helm.sh/docs/intro/install/)
5. Install [PLAS-TESK](https://github.com/PlatformedTasks/PLAS-TESK)
6. Install [PLAS-cwl-tes](https://github.com/PlatformedTasks/PLAS-cwl-tes)

Now you should have a PLAS compatible testbed ready. Continue reading the README's [hands-on example](README.md#Hands-on-example) to try a PLAS example.
