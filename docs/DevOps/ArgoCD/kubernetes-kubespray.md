# Kubernetes
## Overview
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a platform for automating the deployment, scaling, and operations of application containers across clusters of hosts.

Kubernetes is designed to be extensible and portable, allowing users to take advantage of the latest features and technologies. It provides a rich set of APIs for configuring and managing the cluster, and it supports a wide range of container runtimes, storage systems, and networking solutions.

## Architecture
The Kubernetes architecture consists of a master node and multiple worker nodes. The master node is responsible for managing the cluster and its components, while the worker nodes run the applications and workloads.

## Master Node
The master node is the control plane of the Kubernetes cluster. It is responsible for managing the cluster state, scheduling workloads, and maintaining the desired state of the cluster. The master node consists of several components, including the API server, scheduler, controller manager, and etcd.

- **API Server**: The API server is the central management point for the cluster. It exposes the Kubernetes API, which allows users to interact with the cluster and manage its resources.
- **Scheduler**: The scheduler is responsible for placing workloads on the worker nodes. It takes into account factors such as resource requirements, quality of service, and affinity/anti-affinity rules to make intelligent decisions about where to run the workloads.
- **Controller Manager**: The controller manager is responsible for maintaining the desired state of the cluster. It runs a set of controllers that watch the cluster state and take action to ensure that the actual state matches the desired state.
- **etcd**: etcd is a distributed key-value store that is used to store the cluster state. It provides a reliable and consistent way to store and retrieve data, and it is used by the master components to store configuration data, state, and metadata.
## Worker Node
The worker nodes are responsible for running the applications and workloads. They are managed by the master node and communicate with it to receive instructions and updates. Each worker node runs a container runtime, such as Docker or containerd, and a kubelet agent that interacts with the master node.

- **Container Runtime**: The container runtime is responsible for running the containers on the worker node. It pulls the container images from a registry, creates the containers, and manages their lifecycle.
- **Kubelet**: The kubelet is an agent that runs on each worker node and communicates with the master node. It is responsible for managing the containers on the node, including starting, stopping, and monitoring them.
## Installation
We use ansible to install and configure the Kubernetes cluster. The Kubespray project provides a set of Ansible playbooks for deploying a production-ready Kubernetes cluster. It automates the installation and configuration of the cluster components, including the master node, worker nodes, and networking and storage solutions.

## Prerequisites
Before installing Kubernetes, make sure you have the following prerequisites in place: - A set of servers to use as master and worker nodes. The servers should meet the minimum hardware requirements for running Kubernetes. The master nodes count should be odd to ensure high availability. - A user with sudo privileges on the servers. - A working SSH connection to the servers. - Internet access on the servers to download the required packages and container images.

## Installation Steps
To install Kubernetes using Neor documentation, follow these steps: 1. Create inventory repository and create a hosts.yaml file with the details of the master and worker nodes. An example:

```
all:
  vars:
    # The ssh key path that will copy to the remote servers in the securing phase.
    # It will generate if it does not exist.
    ssh_key_path: /root/.ssh/[name]_vms
    ssh_config_dir: /root/.ssh
    ssh_config_path: "{{ ssh_config_dir }}/config"
    # these variables will be set in .ssh/config file.
    # they are not used to ssh into anywhere
    ssh_user_initial: ubuntu
    ssh_user_desired: ansible
    ssh_port_initial: 22
    ssh_port_desired: 23389
    # this can have two values: initial / desired
    current_state: initial
    # The ssh key that will set in the .ssh/config file. And further used to ssh into the remote servers.
    ssh_identity_file: /root/.ssh/[name]_vms
    private_range: "192.168.1.0/24"
    apt_proxy_server: proxy.sample
    apt_proxy_port: 12889
  hosts:
    [name]-stage-k8s-lb1:
      ansible_host: localhost
      ansible_connection: local
      ansible_python_interpreter: "{{ansible_playbook_python}}"
    [name]-stage-k8s-master1:
      ansible_host: [name]-stage-k8s-master1
      ip: 172.16.0.177
      access_ip: 172.16.0.177
    [name]-stage-k8s-master2:
      ansible_host: [name]-stage-k8s-master2
      ip: 172.16.0.178
      access_ip: 172.16.0.178
    [name]-stage-k8s-master3:
      ansible_host: [name]-stage-k8s-master3
      ip: 172.16.0.179
      access_ip: 172.16.0.179
    [name]-stage-k8s-worker1:
      ansible_host: [name]-stage-k8s-worker1
      ip: 172.16.0.180
      access_ip: 172.16.0.180
    [name]-stage-k8s-worker2:
      ansible_host: [name]-stage-k8s-worker2
      ip: 172.16.0.181
      access_ip: 172.16.0.181
    [name]-stage-k8s-worker3:
      ansible_host: [name]-stage-k8s-worker3
      ip: 172.16.0.182
      access_ip: 172.16.0.182
  children:
    kube_control_plane:
      hosts:
        [name]-stage-k8s-master1:
        [name]-stage-k8s-master2:
        [name]-stage-k8s-master3:
    kube_node:
      hosts:
        [name]-stage-k8s-worker1:
        [name]-stage-k8s-worker2:
        [name]-stage-k8s-worker3:
    etcd:
      hosts:
        [name]-stage-k8s-master1:
        [name]-stage-k8s-master2:
        [name]-stage-k8s-master3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
    kube_lb:
      hosts:
        [name]-stage-k8s-lb1:
      vars:
        ingress_controller_http_nodeport: 31080
        ingress_controller_https_nodeport: 31443
```

### Installing ansible
- sudo apt install python3-pip python3-venv
- mkdir .venv   ( in home directory , not in project folder )
- python3 -m venv ~/.venv/ansible ( or any other name )   ( if we use python3 -m venv --copies ~/.venv/ansible , the python will not be symbolic link to python of system , it will copy all the env )
- source the env ( what is the difference between run a bash script in a regular way or source it ? if we source a bash script , it will run in the bash that it exists , we can source with . filename.sh ) ==we source the env with source ~/.venv/ansible/bin/activate==
- pip install ansible

## Follow these steps to set up the necessary requirements for executing the playbooks
- apt install python3-venv
- python3 -m venv venv
- source venv/bin/activate
- pip install -r requirements.txt

dont forget to get the pubkey from private key

```ssh-keygen -y -f /path/to/private-key > /path/to/public-key```

---
### If you are using cilium, follow this steps for the ipsec-key
for generate ipsec-encryption: https://github.com/kubernetes-sigs/kubespray/blob/master/docs/cilium.md#ipsec-encryption

---
1. Run the ```ssh-config-generator``` playbook to generate the ssh config file.
2. Test the ssh connection to the remote servers. You can use the following command to test the ssh connection:

```ansible -i hosts.yaml -m ping all```

ssh-keygen -f ~/.ssh/id_rsa -y > ~/.ssh/id_rsa.pub
generate pubkey from private key

1. Run the ```securing``` playbook to secure the remote servers.
2. Change the ```current_state``` variable to ```desired``` in the ```hosts.yaml``` file.
3. Run the ```ssh-config-generator``` playbook to generate the ssh config file with the desired state.
4. Test the ssh connection to the remote servers. You can use the following command to test the ssh connection:

```ansible -i hosts.yaml -m ping all```

1. Clone the Kubespray repository and checkout the desired version.
2. Copy the inventory/sample/patch and inventory/sample/group_vars directories to the inventory repository.
3. Update the k8s cluster configuration in group_vars directory.
4. Run the cluster.yml playbook of Kubespray to install and configure the Kubernetes cluster.
5. Its better to run this command in tmux
```
ansible-playbook -i hosts.yaml --become --become-user=root cluster.yml
```
## Verification
After the installation, ssh into the master node and run the following commands to verify the installation:
```
kubectl get nodes -o wide
kubectl get pods -n kube-system -o wide
```
You should see all the nodes present and ready, and the kube-system pods running.