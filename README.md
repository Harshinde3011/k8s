# k8s components

![k8s-components](https://user-images.githubusercontent.com/108318918/218641410-1f708fdc-bab7-4eeb-aa5e-e0c2bb4c71e2.jpg)

# K8S cluster set-up with containerd
System Specification:

OS: Ubuntu 20.04 LTS server
RAM: 4GB or more
Disk: 30GB+
CPU: 2 core or more
Swap Memory: Diable
Two nodes: VirtualBox was used to launch both nodes(Master Hostname: controlplane and Worker Hostname: computeplaneone)
Network: Bridge was used to have internal and external access (if not we can use 2 interface, one for NAT and one as host-only adaptor)
common tasks/commands to execute on all the nodes
Docker and Containerd containerdv2.0

# Execute below commands on both "Master" and "Worker"
### disable swap
sudo swapoff -a

### Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

### sysctl params required by setup, params persist across reboots
#cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
#net.bridge.bridge-nf-call-iptables  = 1
#net.bridge.bridge-nf-call-ip6tables = 1
#net.ipv4.ip_forward                 = 1
#EOF

### Apply sysctl params without reboot
sudo sysctl --system

### Install Containerd
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y

### Create containerd configuration
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

### Edit /etc/containerd/config.toml
sudo nano /etc/containerd/config.toml
SystemdCgroup = true
sudo systemctl restart containerd

### Add Kubernetes APT repository and install required packages
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet="1.29.0-*" kubectl="1.29.0-*" kubeadm="1.29.0-*"
sudo apt-get update -y
sudo apt-get install -y jq

sudo systemctl enable --now kubelet
sudo systemctl start kubelet

# Commands Only on Master
sudo kubeadm config images pull

sudo kubeadm init

mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config

### Nwtwork plugin = calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml

kubeadm token create --print-join-command

Copy Token 


# Commands on Worker

sudo kubeadm reset pre-flight checks

paste token
