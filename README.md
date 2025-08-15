# Kubernetes The Hard Way Using Ansible

This Ansible playbook is inspired by [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) and is designed to deploy a Kubernetes cluster. While kubernetes-the-hard-way focuses on creating a minimalistic cluster for educational purposes, this playbook extends it by adding more features to make the cluster production-ready.

## Key Features:
* Supports multiple **Ansible runs** without causing issues in the cluster
* Enables **upgrading** and **downgrade** the Kubernetes cluster to a new version with zero downtime
* Allows **adding new nodes** to the cluster without disrupting workloads
* Deploys using **hostnames** instead of static IP addresses, improving flexibility
* Utilizes **kube-vip** to provide high availability for the control plane
* Uses **Cilium** as the CNI, eliminating the need for kube-proxy for improved networking
* Fully **offline deployment** to prevent 403 errors and external dependency issues
* Significantly **faster** installation compared to kubeadm and kubespray

## Preparation
Before using the Ansible playbooks, you'll need to perform the following preparation steps:

1. **Create the Necessary VMs**:
   - Set up the required number of virtual machines (VMs) for your Kubernetes cluster. Ensure you have a mix of `controller` and `worker` nodes, with an odd number of controllers being preferable for fault tolerance. For example Three VMs based on Debian or CentOS for the Kubernetes cluster (Debian 12 recommended)
   - One VM with Docker for deploying a **Harbor registry** and **dnsmasq**

2. **Clone the Project**:
   - Clone this repository to the `/opt/kube-ansible` directory on your `deployer` server:

   ```bash
   git clone https://github.com/RezaBabapour/kube-ansible.git /opt/kube-ansible
   ```

3. **Install Ansible**:
  - Install Ansible version 2.12 or greater on the first Kubernetes VM.
  - Install sshpass all Kubernetes VMs
   ```
   apt install ansible
   apt install sshpass
   ```

4. **Populate the inventory File and group_vars**:
   - There is a Sample file in ansible dir `inventory/sample.yml`, make a copy from it and populate it with your own values.
   - in `group_vars` there is a `sample` dir, make a copy from it and populate it with your own values.


## Deploy Cluster

1. Specify `kubernetes.version` in `group_vars/all.yml`
2. Create `files` dir in project root for keeping needed binary files during the cluster setup. Download needed binaries and store them in recpective directory. Find compatible binary versions from Kubernetes changelog. Here is the `files` directory structure:
```bash
files/
├── cfssl_1.6.5_linux_amd64
├── cfssljson_1.6.5_linux_amd64
├── cni-v1.4.0
│   ├── bandwidth
│   ├── bridge
│   ├── dhcp
│   ├── dummy
│   ├── firewall
│   ├── host-device
│   ├── host-local
│   ├── ipvlan
│   ├── loopback
│   ├── macvlan
│   ├── portmap
│   ├── ptp
│   ├── sbr
│   ├── static
│   ├── tap
│   ├── tuning
│   ├── vlan
│   └── vrf
├── containerd-v1.7.13
│   ├── containerd
│   ├── containerd-shim
│   ├── containerd-shim-runc-v1
│   ├── containerd-shim-runc-v2
│   ├── containerd-stress
│   └── ctr
├── etcd-v3.5.11
│   ├── etcd
│   ├── etcdctl
│   └── etcdutl
├── helm-v3.17.0
├── nerdctl-v2.0.3
│   └── nerdctl
├── runc-v1.1.12
│   └── runc
└── v1.30.9
    ├── kube-apiserver
    ├── kube-controller-manager
    ├── kubectl
    ├── kubelet
    └── kube-scheduler
```

1. Now you can execute the `deploy-cluster.yml` playbook:
```bash
ansible-playbook -i inventory/simurgh.yml -kK -e ansible_user=user -e install=1 deploy-cluster.yml
```

2. Add new nodes
```bash
ansible-playbook -i inventory/simurgh.yml -kK -e ansible_user=reza -e add_new_node=true deploy-cluster.yml --tags=worker
```
## TODO

# create snapshot
etcdctl snapshot save snapshot.db --endpoints=https://10.10.10.10:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem

# check snapshot
etcdutl --write-out=table snapshot status snapshot.db

etcdctl --write-out=table endpoint status --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem --endpoints=https://k8s-129.simurgh.academy:2379,https://k8s-130.simurgh.academy:2379,https://k8s-131.simurgh.academy:2379


curl --cacert /var/lib/kubernetes/ca.pem \
     --cert /var/lib/kubernetes/kubernetes.pem \
     --key /var/lib/kubernetes/kubernetes-key.pem \
     -H "Content-Type: application/json-patch+json" \
     -X PATCH https://127.0.0.1:6443/api/v1/nodes/appsdfpspsd0101/status \
     --data "[{ \"op\": \"remove\", \"path\": \"/status/conditions/0\"}]"
