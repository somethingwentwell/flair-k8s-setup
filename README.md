# flair-k8s-setup

## Create Disk

```
export DISK_NAME=test-disk1
qemu-img create -f raw /var/lib/libvirt/images/${DISK_NAME}.qcow2 200G
```

## Create VM

```
export VM_NAME=test-vm1
sudo virt-install --name $VM_NAME --os-variant ubuntu20.04 --vcpus 4 --ram 4096 --location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/ --network bridge=virbr0,model=virtio --graphics none --extra-args='console=ttyS0,115200n8 serial' --disk /var/lib/libvirt/images/${DISK_NAME}.qcow2
```

## Assign vGPU to VM
```
virsh shutdown ${VM_NAME}
virsh edit ${VM_NAME}
```

```
  <hostdev mode='subsystem' type='mdev' model='vfio-pci'>
    <source>
      <address uuid='4f28ef3b-6710-449d-9df0-84cf3ca29308'/>
    </source>
  </hostdev>
```
```
virsh start ${VM_NAME}
```

## Check VM IP

```
virsh net-dhcp-leases default
export VM_IP=<the VM IP>
```

## Install Nvidia driver in VM

Disable nouveau driver
```
ssh ubuntu@${VM_IP}
sudo vi /etc/modprobe.d/blacklist-nouveau.conf
```

add below config in the text editor
```
blacklist nouveau
options nouveau modeset=0
```

Update and reboot
```
sudo update-initramfs -u
sudo reboot
lsmod | grep nouveau
```

Install & verify Nvidia drivers
```
wget https://storage.googleapis.com/nvidia-drivers-us-public/GRID/vGPU13.5/ NVIDIA-Linux-x86_64-470.161.03-grid.run
chmod +x NVIDIA-Linux-x86_64-470.161.03-grid.run
nvidia-smi
```


## Set up Kubernetes clusters

***This is a tailor-made Kubespray for private kvm cluster. Use the Kubespray config in this repo instead of the default/main branch from https://github.com/kubernetes-sigs/kubespray***

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
ansible-playbook -i inventory/local/hosts.ini  cluster.yml -b -v --private-key=~/.ssh/id_rsa -K
```

3. Connect to the clusters
```
export KUBECONFIG=inventory/local/artifacts/admin.conf
kubectl get node
```
