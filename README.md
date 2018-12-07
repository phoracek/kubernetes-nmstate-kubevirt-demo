# kubernetes-nmstate demo

### Installation

Prepare one VM node

```shell
vagrant up
```

Install Kubernetes

```shell
./install-kubernetes
```

Install KubeVirt

```shell
./install-kubevirt

# test:
./kubectl get pods -n kubevirt
./kubectl create -f examples/simple-vmi.yml
./kubectl get pods
./kubectl delete -f examples/simple-vmi.yml
```

Install Multus

```shell
./install-multus

# test:
./kubectl get pods -n kube-system
```

Install OVS-CNI

```shell
./install-ovs-cni

# test:
./kubectl get pods -n kube-system
./kubectl get nodes -o yaml
vagrant ssh node1 -c "sudo ovs-vsctl add-br br100"
./kubectl create -f examples/multus-vmi.yml
./kubectl get pods
./kubectl delete -f examples/multus-vmi.yml
vagrant ssh node1 -c "sudo ovs-vsctl del-br br100"
```

Install nmstate

```shell
./install-nmstate

# test:
./kubectl get pods -n nmstate-default
./kubectl get nodenetworkstates -n nmstate-default
```

### Define network attachments

Now let's create a network defintion for an OVS bridge

```shell
export PS1="\\$\[$(tput sgr0)\] "
clear
vi examples/network-attachment.yml
```

Then create it

```shell
./kubectl create -f examples/network-attachment.yml

# test:
./kubectl get network-attachment-definition blue-network -o yaml
```

### Connect a VM to network using this network attachment

Now let's create a VM that requests an OVS bridge

```shell
vi examples/vmi.yml
```

And create it

```shell
./kubectl create -f examples/vmi.yml

# test:
./kubectl get vmi test-vmi -o wide
pod=$(./kubectl get pods | grep test-vmi | cut -d ' ' -f 1)
./kubectl get pods
./kubectl describe pod $pod
```

Whoops, it cannot be scheduled, there is no such bridge available on any node

### Use nmstate to get node network configuration

Let's use NMState to see what is wrong

```shell
./kubectl get nodenetworkstates -n nmstate-default -o yaml | less
```

Point to one extra interface eth2, we will use that for secondary L2 connection

### Configure OVS bridge on the node

First we have to configure OVS bridge on top of it

```shell
vi examples/node-network.yml
```

Execute the config

```shell
./kubectl apply -f examples/node-network.yml
```

Show that it was configured

```shell
./kubectl get nodenetworkstates -n nmstate-default -o yaml | less
./kubectl get node node1 -o yaml | less
```

### Was VM created?

Now, let's see if the VM was scheduled

```shell
pod=$(./kubectl get pods | grep test-vmi | cut -d ' ' -f 1)
./kubectl describe pod $pod
./kubectl get vmi test-vmi -o wide
./virtctl console test-vmi
```
