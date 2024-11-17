# Single Node OpenShift Lab for Red Hat OpenShift AI on AWS

## Description
This project is a basic automation to provision a Single Node OpenShift (SNO) on AWS with a standard
installation of Red Hat OpenShift AI and all its dependencies.

By default the SNO is created with the `g4dn.metal` flavor.

After the basic cluster installation the automation can install and configure the following features:
- HTPasswd identity provider
- Node Feature Discovery Operator
- Nvidia GPU Operator
- Serverless Operator
- Service Mesh Operator
- Red Hat OpenShift AI Operator

## Prerequisites
Ansible 2.9+ must be installed on the system. Supported systems are Linux and macOS. 

The following additional packages must be installed:
- Python Netaddr to manipulate addressed (`python3-netaddr` on Fedora/RHEL or `netaddr` if installing pip3)
- Python OpenShift to access OpenShift API (`python3-openshift` on Fedora/RHEL or `openshift` if installing with pip3)
- Python Passlib to generate htpasswd files (`python3-passlib` on Fedora/RHEL or `passlib` if installing with pip3)

A future release will automatically handle those dependencies on the different 
platforms.

The following Ansible Collections are required:
- `community.general`, used by core modules
- `ansible.netcommon`, used to apply IP filtering
- `kubernetes.core`, used to test installation results

To install them:
```
$ ansible-galaxy collection install -r requirements.yml
```

### Cluster deployment
After receiving the informations run the cluster deploy playbook locally:
```
$ ansible-playbook main.yaml
```
  
Ansible will prompt for **sudo** password to install the latest `oc` and `openshift-install` 
binaries under the `/usr/local/bin` path.

Users are expected to provide the following mandatory informations in the `user_info.yaml` file in order to complete the installation.
- `base_domain`: the sandbox base domain that will be used to expose APIs and Ingress
- `aws_access_key_id`: the AWS acces key id available in the received e-mail.
- `aws_secret_access_key`: the AWS secret access key available in the received e-mail.
- `pull_secret`: the Red Hat pull secret necessary to pull the cluster images
- `ssh_key`: the public SSH key that will be injected in the nodes
- `install_dir`: the installation directory where install files and logs will be created

After the installation login to the cluster using the provided informations 
in the Ansible output or in the `.openshift_install.log` file in the installation
directory.

Under `install_dir/auth` the installation deploys the `kubeconfig` file.

### Cluster destroy
The OpenTLC labs use AWS sandboxes for demo and training purposes but it is
important to avoid unnecessary compute resource usage, for cost and 
environmental reasons. After finishing using a cluster destroy it using
the follwing command:

```
$ ansible-playbook destroy.yaml
```

The `install_path` variable must be provided (again, in the `user_info.yaml` file, 
to confirm the previously used installation directory.


### Optional Extra Configs
Users can customize cluster installation configs or day two customizations by 
editing the `cluster_config_vars.yaml`. After the base cluster installation is
complete users can choose to create custom users with the HTPasswd identity 
provider or to deploy common extra operators with a simple boolean variable.
Available extra configs, with their predefined values, are:

```
---
# Cluster name
cluster_name: sno

# Sizing of master nodes
master_flavor: g4dn.metal

# AWS install region
aws_region: us-east-2

# The service network used top allocate service vips. To test Submariner
# multicluster connectivity be sure to use diffrenet CIDRs.
service_network: 172.30.0.0/16

# Enable FIPS mode
fips_enabled: false

## Post Install Configs
# HTPasswd Users. Change passwords before installing.
htpasswd_users:
  developer1: RedHat123!
  developer2: RedHat123!
```

## Maintainers
Gianni Salinetti <gsalinet@redhat.com>  

