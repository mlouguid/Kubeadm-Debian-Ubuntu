# How to Install k8s in Ubuntu/Debian
Learning to install Kubernetes using kubeadm will help you (as a developer) to know the nitty gritty of k8s (kubernetes). Learning k8s tools and container runtime is useful for developing containerised applications and hosting them. Although, there are already k8s installation tools readily available such as Minikube, kind, rancher_k3s. They are all great and fast in installing and getting ready a cluster. However, installing k8s with kubeadm will certainly give you insights that those tools just skips and automates for you.

# Prerequisites
Linux (Ubuntu) host VM. Single node or more than one node.
2 GB of RAM atleast, 2 CPUs or more.
Full network connectivity between machines
Unique hostnames
Port 6443 must be open. Check via
```
nc 127.0.0.1 6443
```
Swap disable using `sudo swapoff -a`
```
sudo swapoff -a
```
Installing a container runtime
To run containers in Pods, Kubernetes uses a container runtime.

By default, Kubernetes uses the Container Runtime Interface (CRI) to interface with your chosen container runtime.

If you don’t specify a runtime, kubeadm automatically tries to detect an installed container runtime by scanning through a list of known endpoints.

If multiple or no container runtimes are detected kubeadm will throw an error and will request that you specify which one you want to use.

See container runtimes for more information.

Install using the apt repository
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

Set up Docker’s apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Install the containerd
```
 sudo apt-get install containerd.io
```
This version for Ubuntu, to install for debian follow this link:
https://docs.docker.com/engine/install/

Install and configure prerequisites
The following steps apply common settings for Kubernetes nodes on Linux.

You can skip a particular setting if you’re certain you don’t need it.

For more information, see Network Plugin Requirements or the documentation for your specific container runtime.

Forwarding IPv4 and letting iptables see bridged traffic
Execute the below mentioned instructions:
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
Verify that the br_netfilter, overlay modules are loaded by running the following commands:
```
lsmod | grep br_netfilter
lsmod | grep overlay
```
Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```
cgroup drivers
On Linux, control groups are used to constrain resources that are allocated to processes.

Both the kubelet and the underlying container runtime need to interface with control groups to enforce resource management for pods and containers and set resources such as cpu/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a cgroup driver. It’s critical that the kubelet and the container runtime use the same cgroup driver and are configured the same.

There are two cgroup drivers available:

cgroupfs
systemd
This command modifies the containerd configuration to use systemd for cgroup management and updates the sandbox image to pause:3.9, then saves the changes to /etc/containerd/config.toml. These adjustments enhance compatibility and performance for Kubernetes clusters .
```
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml
```
Installing kubeadm, kubelet and kubectl
You will install these packages on all of your machines:

kubeadm: the command to bootstrap the cluster.
kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
kubectl: the command line util to talk to your cluster.
Update the apt package index and install packages needed to use the Kubernetes apt repository:
```
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
2. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:
```
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
3. Add the appropriate Kubernetes apt repository. Please note that this repository have packages only for Kubernetes 1.29; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).
```
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
4. Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
5. (Optional) Enable the kubelet service before running kubeadm:
```
sudo systemctl enable --now kubelet
```
Creating a cluster with kubeadm
```
kubeadm init
```
To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:
```
 mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```
$ kubectl get node
NAME       STATUS     ROLES           AGE    VERSION
marouane   NotReady   control-plane   3m9s   v1.29.7
Make Nodes Ready
```
Calico manifests​
Calico can also be installed using raw manifests as an alternative to the operator. The manifests contain the necessary resources for installing Calico on each node in your Kubernetes cluster. Using manifests is not recommended as they cannot automatically manage the lifecycle of the Calico as the operator does. However, manifests may be useful for clusters that require highly specific modifications to the underlying Kubernetes resources.

Install Calico with Kubernetes API datastore, 50 nodes or less​
Download the Calico networking manifest for the Kubernetes API datastore.
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml -O
```
If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR with kubeadm, no changes are required — Calico will automatically detect the CIDR based on the running configuration. For other platforms, make sure you uncomment the CALICO_IPV4POOL_CIDR variable in the manifest and set it to the same value as your chosen pod CIDR.
Customize the manifest as necessary.
Apply the manifest using the following command.
```
kubectl apply -f calico.yaml
```
