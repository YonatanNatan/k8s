# k8s
steps to create k8s cluster with 3 master note and 2 worker node 

OS - Ubuntu 24.02
Min Require
2 GiB RAM
2 CPU

# 1.Disable Swap
swapoff -a; sed -i '/swap/d' /etc/fstab

# 2.Disable FireWall
ufw disable

# 3. common settings for Kubernetes nodes 

1.cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
2.overlay
3.br_netfilter
4.EOF

sudo modprobe overlay
sudo modprobe br_netfilter
#sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
#Apply sysctl params without reboot
sudo sysctl --system

# 3.Install Container (Docker engine)
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world

# 4.Installing kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# 5.Creating a cluster with kubeadm
kubeadm init --control-plane-endpoint="192.168.153.102:6443" --upload-certs --apiserver-advertise-address=192.168.153.103 --pod-network-cidr=192.168.0.0/16


ERROR

SOLUTION
sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd 
