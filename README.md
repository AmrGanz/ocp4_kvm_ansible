# ocp4_kvm_ansible
- This Ansible playbook will install OCP4 on a KVM environment.

# Lab Environment, I have tested the playbooks on:
- Lenovo Thinkpad P70 Laptop
  - Core I7 vPro 8th Gen
  - 64 GB RAM
- RHEL 7.7 "minimum installation" + ansible-2.9.6

# Tested OCP versions
- OCP 4.2.12
- OCP 4.2.18
- OCP 4.3.0
- OCP 4.3.13
- OCP 4.4.3
- OCP 4.4.5

# What do you have to do beforehand:
- The playbooks will try to download everything that you will need under "/var/www/html/downloads/"
- Make sure you are using "root" user
- make sure your system "bastion host" is subscribed or at least has a configured repositories to install the following packages [playbooks will try to install it]:
```
- httpd
- dnsmasq
- qemu-kvm
- libvirt
- libvirt-python
- libguestfs-tools
- virt-install
- libvirt-daemon-driver-network
- virt-manager
```
- And of cousre you will need to install Ansible on the bastion host :)

# Variables to set before starting the playbook:

- If you set **ocpversion** parameter using "-e" option, the playbook will check if this versions is a valid version and start installing it.

- If you are not sure which OCP versions are avaiable, don't set this parameter and the playbook will give you a list of the available versions to choose from, for example:
```
- The playbook will ask you to select between three OCP minor vrsions:
- 4.1
- 4.2
- 4.3

- Example:

TASK [Listing available minor versions] ************************************************************************************************
ok: [localhost] => {
    "minorversions.stdout_lines": [
        "4.1", 
        "4.2", 
        "4.3"
    ]
}
```
- Then it will show all of the available micro versions to select from, for example and fater selecting "4.3" from the previous output:
```
TASK [debug] ***************************************************************************************************************************
ok: [localhost] => {
    "microversions.stdout_lines": [
        "4.3.0", 
        "4.3.1", 
        "4.3.2", 
        "4.3.3", 
        "4.3.5", 
        "4.3.8", 
        "4.3.9", 
        "4.3.10", 
        "4.3.11", 
        "4.3.12", 
        "4.3.13", 
        "4.3.14", 
        "4.3.15", 
        "4.3.17"
    ]
}

```
- Other variables that you need to be set:
```
# Pull secret, it can be grabbed from here "https://cloud.redhat.com/openshift/install/metal/user-provisioned":
pullsecret: 'PASTE HERE'

# bastion_ip, is the IP of the host you are running Ansible playbook on:
bastion_ip: 192.168.1.142

# Base MAC address "first 5 segments", the playbook will complete the MAC address for each node:
macbase: "90:00:00:00:00:"

# This octedt will be added to "192.168.XXX.0" network that will be dedictaed for cluster nodes:
network_subnet_octet: 125

# Bootstrap VM resources configuration:
bootstrap_cpu: 2
bootstrap_mem: 8192
bootstrap_disk: 50

# Loadbalancer VM resources configuration:
loadbalancer_cpu: 1
loadbalancer_mem: 1024

# Masters VMs count and resources configuration:
master_count: 3
master_cpu: 4
master_mem: 16384
master_disk: 100

# Workers VMs count and resources configuration:
worker_count: 3
worker_cpu: 2
worker_mem: 8192
worker_disk: 50

# VMs disks location on the system, the default is the following value:
vms_disks_location: "/var/lib/libvirt/images/"

# Cluster name and domain name:
domain_name: mydomain.com
cluster_name: ocp43

# While using "destroy.yaml", if you set the following parameter to "true" all downloaded files and generated ones will be deleted:
delete_downloads: false
```


# Which playbook to run:
- There are two main playbooks [tart_point.yaml and destroy.yaml], and you can use them as follows:
- start_point.yaml [Interactive way]:
```
# ansible-playbook start_point.yaml
- The playbook will list the avilabl OCP4 minor and micro versions to select from it manually before it proceeds installing the cluster componenets.
```
- start_point.yaml [Automatic way]:
```
# ansible-playbook start_point.yaml -e ocpversion=4.3.13
- The playbook will proceed to donwload and install the required OCP version "4.3.13 in this example".
```
- destroy.yaml [Destroy OCP4 cluster without deleting downloaded files]
```
# ansible-playbook destroy.yaml
- Destroy "delete" all of the configured VMs and services configurations "will not delete what has been downloaded to save time"
```
- destroy.yaml [Destroy OCP4 cluster and delete downloaded files]
```
# ansible-playbook destroy.yaml -e delete_downloads=true
- Pass the parameter "delete_downloads=true" to also delete all of downloaded files "/var/www/html/downloads/ directoy will be deleted"
```
# Detailed Steps:
- Make sure your are using the "root"
```
[bastionhost]# sudo su -
```
- Download project files:
```
[bastionhost]# git clone https://github.com/AmrGanz/ocp4_kvm_ansible.git
```
- Switch to the directory:
```
[bastionhost]# cd ocp4_kvm_ansible
```
- Make sure you have enough reources, specially in "/var":
  - /var/www/html/downloads/ will take around 2GB
  - /var/lib/libvirt/images/ will depend on how much disk space you gave to each VM

- Edit the variables in `ocp4_kvm_ansible/vars/cluster_variables` to better suit your setup "please give enough resources to master and worker nodes"

- Start the installation process:
```
[bastionhost]# ansible-playbook start_point.yaml -e ocpversion=4.3.13
```
- If the downloads are complete but the installation failed, please delete the environment then repeat the same process but skip the "downloading" task this time:
```
[bastionhost]# ansible-playbook destroy.yaml
[bastionhost]# ansible-playbook start_point.yaml --skip-tags download
```
- If the playbook completed successfilly, you should see something similar to the follwoing:
```
TASK [Cluster access details] **********************************************************************************************************
Tuesday 26 May 2020  00:32:47 +0400 (0:00:00.245)       0:22:38.901 *********** 
ok: [localhost] => {
    "msg": [
        "Cluster version:           4.3.13", 
        "Cluster name:              ocp44", 
        "Cluster domain:            mydomain.com", 
        "API URL:                   api.ocp44.mydomain.com:6443", 
        "Console URL:               console-openshift-console.apps.ocp44.mydomain.com", 
        "Kubeadmin credentials:     kubeadmin/YYWEi-Y9dwm-CnKLs-V5W8L", 
        "Cluster Auth files:        /var/www/html/downloads/4.3.13/install_dir/auth/kubeconfig", 
        "Certificate based access:  export KUBECONFIG=/var/www/html/downloads/4.3.13/install_dir/auth/kubeconfig"
    ]
}
```
- You can check if the OCP4 deployment is completed "or follow deployment progress" by connecting to the "bootstarp" VM and running the following commands "example":
```
[bastionhost]# ssh -i ./ocp-ssh-key core@bootstrap.ocp4.mydomain.com
[core@bootstrap ~]$ sudo su -
Last login: Fri Apr 10 09:22:28 UTC 2020 on pts/0
[root@bootstrap ~]# journalctl -b -f -u bootkube.service
...
Apr 10 09:56:41 bootstrap.ocp4.mydomain.com bootkube.sh[1601]: Skipped "cluster-network-01-crd.yml" customresourcedefinitions.v1beta1.apiextensions.k8s.io/networks.operator.openshift.io -n  as it already exists
Apr 10 09:56:45 bootstrap.ocp4.mydomain.com bootkube.sh[1601]: Skipped "cluster-network-02-config.yml" networks.v1.config.openshift.io/cluster -n  as it already exists
Apr 10 09:56:46 bootstrap.ocp4.mydomain.com bootkube.sh[1601]: Skipped "cluster-proxy-01-config.yaml" proxies.v1.config.openshift.io/cluster -n  as it already exists
Apr 10 09:56:47 bootstrap.ocp4.mydomain.com bootkube.sh[1601]: Tearing down temporary bootstrap control plane...
Apr 10 09:56:51 bootstrap.ocp4.mydomain.com bootkube.sh[1601]: bootkube.service complete
```
- Once you have confirmed that OCP4 deployment is done, login to OCP4 cluster and confirm that the nodes are running:
```
[bastionhost]# export KUBECONFIG=/var/www/html/downloads/4.3.13/install_dir/auth/kubeconfig
[bastionhost]# /var/www/html/downloads/4.3.13/oc get nodes
NAME                         STATUS   ROLES           AGE   VERSION
master-1.ocp4.mydomain.com   Ready    master,worker   42m   v1.14.6+c07e432da
master-2.ocp4.mydomain.com   Ready    master,worker   41m   v1.14.6+c07e432da
master-3.ocp4.mydomain.com   Ready    master,worker   41m   v1.14.6+c07e432da
worker-1.ocp4.mydomain.com   Ready    worker          42m   v1.14.6+c07e432da
worker-2.ocp4.mydomain.com   Ready    worker          41m   v1.14.6+c07e432da
worker-3.ocp4.mydomain.com   Ready    worker          42m   v1.14.6+c07e432da
```
- You can delete "bootstrap" VM once the deployment is completed and confirmed.

# Destroy The envirnment:
- To delete all of the created resources "VMs, DNS records, KVM networks,...." run the following playook "**it will NOT delete donwloaded files**":
```
[bastionhost]# ansible-playbook destroy.yaml
```
- To delete all of the downloaded/created resources beside the downloaded files "/var/www/html/downloads/ directoy will be deleted":
```
[bastionhost]# ansible-playbook destroy.yaml -e delete_downloads=true
```

# Notes:
- This is a work in progress, so there might be some bugs here and there and I will keep fixing them.
- Currently, I rely on "command" module to create the required VMs "not ideal I know"
- If the playbook gave an error while creating any of the VMs and failed, you have to delete them all "VMs" then start the process again.


# Known Issues:
- Make sure to give enough resources to masters nodes or you will find number of pods unable to start
- If image-registry operator is not runnung:
~~~
image-registry                                       False	 True          False	  16h
~~~
- In [OCP 4.3](https://docs.openshift.com/container-platform/4.3/installing/installing_bare_metal/installing-bare-metal-network-customizations.html#registry-removed_installing-bare-metal-network-customizations), you must patch the "OpenShift Image Registry Operator" to be in "Managed":
~~~
# oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
~~~
- Make sure to set the storage backend, at least to EmptyDir:
~~~
# oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
~~~

# Current Limitations:
- I have tried the playbboks while using HDD disks and it showed a poor performance, I recommend using SSD to host VMs virtual disks.
- If a task failed, I have to destroy the environment "destroy.yaml" and deploy it again.
