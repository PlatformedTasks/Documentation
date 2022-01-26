# Step by Step
This is a step by step reference to properly configure a working PLAS compatible testbed.

### Kubernetes
* Install a local Kubernetes cluster, for example using Kubeadm as described in the [official page](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* Install an NFS provisioner. We have chosen the NFS Ganesha server as described in the [official guide](https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs). In particular from the [`kubernetes-sigs/nfs-ganesha-server-and-external-provisioner`](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner) we have installed only the deployment and the relative rbac (using the code below), while for the storage class will be installed using the Helm Chart of the TESK-API:

```console
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

* Install and configure the FTP server.
* Install [Helm](https://helm.sh/docs/intro/install/)

Clone the [PLAS-TESK](https://github.com/PlatformedTasks/PLAS-TESK) repository
``` bash
git clone git@github.com:PlatformedTasks/PLAS-TESK.git
```

### TESK-API
1. Dalla documentazione ufficiale di Kubernetes sulle storage class abbiamo scelto di utilizzare [NFS](https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs), in particolare [NFS Ganesha server and external provisioner](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner). Abbiamo creato:
   * Deployment
   * ClusterRole 
   * ClusterRoleBinding
   * Role
   * RoleBinding
   * La storage class l'abbiamo momentaneamente lasciata perdere perché verra installata negli step successivi tramite Helm insieme al TESK-API. E' possibile installare una storageClass provvisoria per testare il funzionamento del provisioner, ma bisogna poi ricordarsi di rimuoverla.
2. Installare e configurare un server [FTP]()
3. Installare [Helm](https://helm.sh/docs/intro/install/)
4. Clone the [PLAS-TESK](https://github.com/PlatformedTasks/PLAS-TESK) repository
``` bash
git clone git@github.com:PlatformedTasks/PLAS-TESK.git
```
6. Entrare nella cartella [tesk](/charts/tesk) ed editare il file [values.yaml](/charts/tesk/values.yaml)
``` yaml
host_name: "10.0.0.10" # FQDN to expose the application, we use the master-node IP address
clusterType: kubernetes
storage: mystorage # Specify the storage type, in particular deploy a storage class named example-nfs used also by the provisioner

...

transfer:
    # If you want local file systems support (i.e. 'file:' urls in inputs and outputs),
    # you have to define these 2 properties.
    active: true
    wes_base_path: '/tmp'
    tes_base_path: '/transfer'    
    pvc_name: 'transfer-pvc' # This PVC must be manually deployend before installing the TESK-API

...

service:
    type: "NodePort" # Expose API directly
    node_port: "31567" 

ftp:
    classic_ftp_secret: ftp-secret # choose the classic methods to providing the credentials
    hostip: 10.0.0.10 # IP of FTP Server, for us is the kubernetes master-node

...
```
5. Create a `secrets.yaml` file with the `username` and `password` of the FTP account

``` yaml
 ftp:
   username: <username>
   password: <password>
 ```

8. Installare il TESK-API tramite helm e verificare che il deployment abbia funzionato correttamente
```bash
$ helm upgrade --install tesk-release . -f secrets.yaml -f values.yaml
Release "tesk-release" does not exist. Installing it now.
NAME: tesk-release
LAST DEPLOYED: Wed Dec 22 18:09:57 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
$           
$ helm list -n tesk
NAME	        NAMESPACE  REVISION	UPDATED                                 	STATUS  	CHART     	APP VERSION
tesk-release	default	     1      	2021-12-20 13:24:34.08857551 +0100 CET	deployed	tesk-0.1.0	dev 
```

### cwl-TES
1. Clone the [PLAS-cwl-tes](https://github.com/PlatformedTasks/PLAS-cwl-tes.git) repository
2. Install the project requirements. To do so, enter the project's root directory and run the command:
```bash
pip3 install -r requirements.txt
```
3. At this point we are ready to create our first CWL task
```yaml
# helm-horovod.cwl.yml
cwlVersion: v1.0
class: CommandLineTool
doc: "helm horovod"
requirements:
  - class: HelmRequirement
    chartRepo: "https://platformedtasks.github.io/PLAS-charts/charts"
    chartVersion: "3.0.0"
    chartName: "horovod"
    executorImage: "platformedtasks/horovod:latest"

inputs:
  - id: train
    type: File
    doc: "original content"
    inputBinding:
      position: 1
  - id: values
    type: File

outputs:
  - id: output
    type: stdout

stdout: horovod

baseCommand: ["python3"]
arguments: ["/horovod/examples/horovod-executor.py", "mpirun -np 2 --mca orte_keep_fqdn_hostnames t --allow-run-as-root --display-map --tag-output --timestamp-output"]
```
4. Submit the task
``` bash
python cwl-tes.py --remote-storage-url ftp://10.0.0.10/files/out --insecure --tes http://10.0.0.10:31567 --leave-outputs tests/helm-horovod.cwl.yml tests/inputs.json
```
## Demo
![](demo_horovod.gif)