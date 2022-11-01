##### Kubernetes Vanilla Installation

The installation covers 2 node Kubernetes cluster:
- napp-k8s-cp-1: the control node
- napp-k8s-wn-1: the worker node

##### Control Node Bootstrap
###### Ubuntu Focal Fossa


````bash
sudo hostnamectl set-hostname napp-k8s-cp-1
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install ntp -y && sudo systemctl enable ntp && sudo systemctl start ntp

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay && sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update && sudo apt-get install -y containerd.io

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl enable containerd && sudo systemctl restart containerd

sudo swapoff -a
# make the option permanent by commenting the swap line in the /etc/fstab

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
chmod ugo+x runc.amd64
sudo chown root:root runc.amd64
sudo mv runc.amd64 /usr/local/sbin/runc

sudo apt-get update && sudo apt-get install -y kubelet=1.24.7-00 kubeadm=1.24.7-00 kubectl=1.24.7-00
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl disable apparmor && sudo systemctl stop apparmor
sudo systemctl enable kubelet
````

````bash
# 10.200.1.9 is the address which can be used by the worker node to reach the Kubernetes API on the control node 
echo "10.200.1.9 napp-k8s-cp-1.cloud.7357.link" | sudo tee --append /etc/hosts
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.24.7 --upload-certs --control-plane-endpoint napp-k8s-cp-1.cloud.7357.link

````

````bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info
````

Add Antrea CNI
````bash
kubectl apply -f https://raw.githubusercontent.com/antrea-io/antrea/main/build/yamls/antrea.yml
````

##### Worker Node Bootstrap
###### Ubuntu Focal Fossa

````bash
sudo hostnamectl set-hostname napp-k8s-wn-1
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install ntp -y && sudo systemctl enable ntp && sudo systemctl start ntp

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay && sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update && sudo apt-get install -y containerd.io

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl enable containerd && sudo systemctl restart containerd

sudo swapoff -a
# make the option permanent by commenting the swap line in the /etc/fstab

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
chmod ugo+x runc.amd64
sudo chown root:root runc.amd64
sudo mv runc.amd64 /usr/local/sbin/runc

sudo apt-get update && sudo apt-get install -y kubelet=1.24.7-00 kubeadm=1.24.7-00 kubectl=1.24.7-00
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl disable apparmor && sudo systemctl stop apparmor
sudo systemctl enable kubelet
````

On the CP node for each WN do:
````bash
sudo kubeadm token create --print-join-command
````

The given command above, use it to join the WN to the cluster:
````bash
sudo kubeadm join 10.200.1.63:6443 --token jtgblf.m1h85ahizlc578ke \
  --discovery-token-ca-cert-hash sha256:72b4d2f3e6ca30b3c28cfe2a19097bba0283de879b1703f52819eac56a4aab04
````