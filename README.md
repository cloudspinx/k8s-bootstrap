## Role info

> This playbook is not for fully setting up a Kubernetes Cluster.

It only helps you automate the standard Kubernetes bootstrapping pre-reqs.

## Supported OS

- Ubuntu 24.04
- Ubuntu 22.04
- Ubuntu 20.04
= Ubuntu 18.04

## Required Ansible
Ansible version required `2.10+`

## Tasks in the role

This role contains tasks to:

- Install basic packages required
- Setup standard system requirements - Disable Swap, Modify sysctl, Disable SELinux
- Install and configure a container runtime of your Choice - cri-o, Docker, Containerd
- Install the Kubernetes packages - kubelet, kubeadm and kubectl

## How to use this role

- Clone the Project:

```bash
$ git clone https://github.com/jmutai/k8s-pre-bootstrap.git
```

- Configure `/etc/hosts` file in your bastion or workstation with all nodes and ip addresses. Example:

```bash
192.168.200.10 k8smaster01.example.com k8smaster01
192.168.200.11 k8smaster02.example.com k8smaster02
192.168.200.12 k8smaster03.example.com k8smaster03

192.168.200.13 k8snode01.example.com k8snode01
192.168.200.14 k8snode02.example.com k8snode02
192.168.200.15 k8snode03.example.com k8snode03
192.168.200.16 k8snode04.example.com k8snode04
```

- Update your inventory, for example:

```bash
$ vim hosts
[k8snodes]
k8smaster01
k8smaster02
k8smaster03
k8snode01
k8snode02
k8snode03
k8snode04
```

- Update variables in playbook file

```yaml
$ vim k8s-prep.yml
---
- name: Prepare Kubernetes Nodes for Cluster bootstrapping
  hosts: k8snodes
  remote_user: root
  become: yes
  become_method: sudo
  #gather_facts: no
  vars:
    k8s_version: "1.29"                                  # Kubernetes version to be installed
    selinux_state: permissive                            # SELinux state to be set on k8s nodes                 
    timezone: "Africa/Nairobi"                           # Timezone to set on all nodes
    k8s_cni: calico                                      # calico, flannel
    container_runtime: cri-o                             # docker, cri-o, containerd 
    pod_network_cidr: "172.18.0.0/16"                    # pod subnet if using cri-o runtime
    # Docker proxy support
    setup_proxy: false                                   # Set to true to configure proxy
    proxy_server: "proxy.example.com:8080"               # Proxy server address and port
    docker_proxy_exclude: "localhost,127.0.0.1"          # Adresses to exclude from proxy
  roles:
    - kubernetes-bootstrap
```

If you are using non root remote user, then set username and enable sudo:

```bash
become: yes
become_method: sudo
```

To enable proxy, set the value of `setup_proxy` to `true` and provide proxy details.

## Running Playbook

Once all values are updated, you can then run the playbook against your nodes.

**NOTE**: Recommended to disable. if you must enable, a pattern in hostname is required for master and worker nodes:

Check playbook syntax to ensure no errors:

```
$ ansible-playbook --syntax-check k8s-prep.yml -i hosts

playbook: k8s-prep.yml
```

Playbook executed as root user - with ssh key:

```
$ ansible-playbook -i hosts k8s-prep.yml
```

Playbook executed as root user - with password:

```
$ ansible-playbook -i hosts k8s-prep.yml --ask-pass
```

Playbook executed as sudo user - with password:

```
$ ansible-playbook -i hosts k8s-prep.yml --ask-pass --ask-become-pass
```

Playbook executed as sudo user - with ssh key and sudo password:

```
$ ansible-playbook -i hosts k8s-prep.yml --ask-become-pass
```

Playbook executed as sudo user - with ssh key and passwordless sudo:

```
$ ansible-playbook -i hosts k8s-prep.yml --ask-become-pass
```

Execution should be successful without errors:

```
TASK [kubernetes-bootstrap : Reload firewalld] *********************************************************************************************************
changed: [k8smaster01]
changed: [k8snode01]
changed: [k8snode02]

PLAY RECAP *********************************************************************************************************************************************
k8smaster01                : ok=23   changed=3    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
k8snode01                  : ok=23   changed=3    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
k8snode02                  : ok=23   changed=3    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
```

Next check article on bootsrapping k8s control plane: https://computingforgeeks.com/install-kubernetes-cluster-on-rocky-linux-with-kubeadm-crio/
