# Configuring K8S-on-VMware using the variables.ft file

## Introduction

This Terraform project allows you to deploy Kubernetes on vSphere. It uses Kubespray for the deployment and allows you to tweak (some, more are coming is time permits) of the deployment options. It will automatically create VM's on VMware vSphere required to deploy Kubernetes using Kubespray. Then it will download Kubespray and prepare the nodes. Finally it can automatically run Kubespray to deploy Kubernetes, but you can also choose to hold-of with the Kubespray deployment (see the run_kubespray parameter below), so that you can tweak the Kubespray parameters. Once have the correct settings in place for Kubespray you can run Kubespray with a single command (~/run_kubespray.sh) to deploy Kubernetes.

The purpose of this project is to provide a quick Kubernetes environment for training and experimenting. If you're going to deploy a production environment, you're beter of with a more mature Kubernetes distribution or deployment.
  

## How to configure the project
All configurable settings are located in the `variables.tf` file at the root of the project.

### Generic vSphere config variables
Parameter | Example | Description
--------- | ------- | -----------
vcenter_server | 10.1.1.1 | The IP address or DNS name of you vCenter server
user | administrator@vsphere.local | The username you wish to use to connect to the vCenter server
password | password123 | The password used to authenticate to the vCenter server
datacenter | Datacenter | The Datacenter within vCenter to use for provisioning the project 
cluster | Cluster | The Cluster within vCenter to use for provisioning the project
datastore | datastore1 | The VMFS/vVol datastore to use for the provisioning of the VM's
vm_network | VM Network | The name of the network / port group to use as management network (your deployment system needs access to this management network via SSH for the Terraform project to be able to complete)
iscsi_network | iSCSI Network | The name of the network / port group to use for iSCSI storage traffic to an external storage system (if you do not need this, read the remakrs below)

### Global K8S cluster parameters
Parameter | Example | Description
--------- | ------- | -----------
username | k8sadmin | Enter the username you wish to create to logon as administrator to the VM's
timezone | Europe/Amsterdam | Enter the timezone, use any supported timezone (http://manpages.ubuntu.com/manpages/bionic/man3/DateTime::TimeZone::Catalog.3pm.html)
run_kubespray | yes | If you wish to automatically run Kubespray, set to `yes`, use `no` if you'd like to be able to first make some changes to the Kubespray config before starting the deployment
kube_version | default | Set the Kubernetes version you'd like to install using Kubespray
private_key | keys/id_rsa-k8s-on-vmware | Set the full path to the RSA keys to use for authentication. If the keys do not exist, they will be created otherwise the existing key files will be used (if you want to use keys in your homedirectory, don't use `~/.ssh/id_rsa` but use `/Users/user/.ssh/id_rsa` instead).
public_key | keys/id_rsa-k8s-on-vmware.pub |  Set the full path to the RSA keys to use for authentication. If the keys do not exist, they will be created otherwise the existing key files will be used (if you want to use keys in your homedirectory, don't use `~/.ssh/id_rsa.pub` but use `/Users/user/.ssh/id_rsa.pub` instead).

### Admin node config parameters
Parameter | Example | Description
--------- | ------- | -----------
hostname | k8s-adminhost | Specify the hostname of the management host used to administer the cluster
num_cpus | 2 | Number of vCPUs used for the management host
memory | 1024 | Virtual memory used for the management host
disk_size | 20 | Disk size for the management host
mgmt_use_dhcp | no | Set to `yes` is DHCP should be used for the IP assignment of the management host
mgmt_interface_name | ens192 | Enter the interface name for the primary network interface (`ens192` for Ubuntu)
mgmt_ip | 192.168.10.150/24 | The static IP address for the management host (only used if `mgmt_use_dhcp` is set to `no`)
mgmt_gateway | 192.168.10.254 | The default gateway used by the management host (only used if `mgmt_use_dhcp` is set to `no`)
mgmt_dns_servers | 8.8.8.8,8.8.4.4 | The DNS servers IP address used by the management host (only used if `mgmt_use_dhcp` is set to `no`)
template | ubuntu-bionic-18.04-cloudimg | The name of the Ubuntu cloudimage template in vCenter used to deploy the management host

### K8S node config parameters
Parameter | Example | Description
--------- | ------- | -----------
hostname | k8s-node | The prefix of the Kubernetes nodes hostname, which will be followed by the node count (eg. when four nodes are used, a hostname of `k8s-node` would result in `k8s-node1` - `k8s-node4`)
number_of_nodes | 3 | Number of worker nodes in the Kubernetes cluster to deploy, generally 3 or more node should be used
num_cpus | 2 | Number of vCPUs used for each Kubernetes node
memory | 2048 | Virtual memory used for each Kubernetes node
disk_size | 20 | Disk size for each Kubernetes node
template | ubuntu-bionic-18.04-cloudimg | The name of the Ubuntu cloudimage template in vCenter used to deploy the Kubernetes nodes
mgmt_use_dhcp | no | Set to `yes` if DHCP should be used for the IP assignment of the management IP address for the Kubernetes nodes
mgmt_interface_name | ens192 | Enter the interface name for the primary network interface (`ens192` for Ubuntu)
mgmt_subnet | 192.168.10.0/24 | The IP subnet for the Kubernetes cluster nodes (only used if `mgmt_use_dhcp` is set to `no`)
mgmt_startip | 151  | The first IP address used, each node will receive a subsequent IP address, eg. 192.168.10.151 for the fist not and 192.168.10.154 for the forth node (only used if `mgmt_use_dhcp` is set to `no`)
mgmt_gateway | 192.168.10.254 | The default gateway used by the Kubernetes nodes (only used if `mgmt_use_dhcp` is set to `no`)
mgmt_dns_servers| 8.8.8.8,8.8.4.4 | The DNS servers used by the Kubernetes nodes (only used if `mgmt_use_dhcp` is set to `no`)
use_iscsi_interface | yes | If a second network interface is to be configured set to `yes`, otherwise set to `no` and read the notes below
iscsi_use_dhcp | no | Set to `yes` if DHCP should be used for the IP assignment of the iSCSI network of the Kubernetes nodes
iscsi_interface_name | ens224 | Enter the interface name for the iSCSI network interface (`ens224` for Ubuntu)
iscsi_subnet | 172.16.10.0/24 | The IP subnet for the Kubernetes cluster nodes for the iSCSI network interface (only used if `iscsi_use_dhcp` is set to `no`)
iscsi_startip | 151 | The first IP address used for the iSCSI network interface, each node will receive a subsequent IP address, eg. 192.168.10.151 for the fist not and 192.168.10.154 for the forth node (only used if `iscsi_use_dhcp` is set to `no`)

## No iSCSI network
If no second network interface is required for iSCSI network, the second network interface can be disabled for the Kubernetes nodes. To do this execute the followin two steps:
1. Set `use_iscsi_interface` in the `variables.tf` configuration file. 
2. Find and remove the following lines from the file `main.tf`:

```
  # If you don't need a separate iSCSI interface, you should remove this second network_interface part
  network_interface {
    network_id          = data.vsphere_network.iscsi_network.id
  }
```

Save both files and continue with the deployment of the cluster.

