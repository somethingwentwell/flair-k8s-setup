# flair-k8s-setup

## Create Disk

```
export DISK_NAME=test-disk1
qemu-img create -f raw /var/lib/libvirt/images/${DISK_NAME}.qcow2 200G
```

## Create VM

```
export VM_NAME=test-vm1
sudo virt-install --name $VM_NAME --os-variant ubuntu20.04 --vcpus 4 --ram 4096 --location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/ --network bridge=virbr0,model=virtio --graphics none --extra-args='console=ttyS0,115200n8 serial' --disk /var/lib/libvirt/images/test-${DISK_NAME}.qcow2
```

## Check VM IP

```
virsh net-dhcp-leases default
export VM_IP=<the VM IP>
```

## Set up Kubernetes clusters

***This is a tailor-made Kubespray for private kvm cluster. Use the Kubespray config in this repo instead of the default/main branch from https://github.com/kubernetes-sigs/kubespray ***

1. Modify inventory/sample/inventory
```
[all]
node1 ansible_ssh_host= 192.168.122.xx ip= 192.168.122.xx ansible_user=ubuntu ansible_ssh_pass=ubuntu
node2 ansible_ssh_host=192.168.122.xx ip=192.168.122.xx ansible_user=ubuntu ansible_ssh_pass=ubuntu

[kube_control_plane]
node1

[etcd]
node1

[kube_node]
node1
node2

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_nodeF
calico_rr
```

2. Run the playbook

```
ansible-playbook -i inventory/sample/hosts.ini  cluster.yml -b -v --private-key=~/.ssh/id_rsa -K
```

3. Connect to the clusters
```
export KUBECONFIG=inventory/local/artifacts/admin.conf
kubectl get node
```