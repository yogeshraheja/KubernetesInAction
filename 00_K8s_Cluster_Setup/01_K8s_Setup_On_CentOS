# KubernetesSetup

Note: The steps are performed to create 2 node K8s cluster, the servers/instances are created on AWS (t2.medium for master and t2.micro for worker node)

Note: You can create servers/instances on any public cloud, local Virtual Box VM's

# Pre-reqs for Master and Worker (we are using centos 7.6, The steps will remain same for Ubuntu but the commands will be different)

# Checking OS type and kernel levels

uname -a

# Checking memory details

free -m or dmidecode

# Checking CPU details
nproc

# Confirming swap is disbaled

free -m   (also cat /etc/fstab) --> if you see anything with SWAP as FS just comment that line and run swapoff -a 

# Disabling SElinux (This step is required only in case of RedHat and CentOS)

getenforce
setenforce 0
vi /etc/selinux/config     --> change enforcing to permissive and save the file
getenforce

# Changing hostnames

hostnamectl set-hostname thinknyxmaster  (remember just allow two special characters in names one in "." And one is "-")
hostnamectl set-hostname thinknyxworkerone

# As we dont have global DNS to using /etc/hosts files and making sure the server are able to talk to each other

ip addr (find the private ips of both server)
vi /etc/hosts (on both server and add IP to name mapping for both server)
Once done ping worker from master and vice versa

# Open specific ports (We are opening all ports just to make things simple but would recommend to open specific ports as mentioned in the below link)
# https://kubernetes.io/docs/reference/networking/ports-and-protocols/
# Also make sure to open UDP 53 for DNS

Port are all open

# Letting iptables see bridged traffic

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

# Installing CRI-O as a runtime (Kindly adjust the versions as per K8s version)

# Note: Even the RPM's are available for latest versions of CRI-O, kindly refer below link and follow the instructions
# https://github.com/cri-o/packaging/blob/main/README.md#usage

export VERSION=1.28
export OS=CentOS_7

echo $VERSION
echo $OS

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params, these persist across reboots.

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo

curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo

yum repolist 

yum install cri-o

systemctl status crio
systemctl start crio
systemctl enable crio
systemctl status crio

crio --version

# Installation of kubeadm, kubectl and kubelet

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

yum repolist

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet

Note: The kubelet will not start unless and until we complete the bootstrapping of our cluster

rpm -qa | grep -i kube*

# Initializing K8S Cluster (RUN THIS ONLY ON YOUR MASTER NODE)
# RUN THIS ONLY ON YOUR MASTER NODE

kubeadm init

# Once kubeadm init command gets completed successfull, observe the last section of the output which will ask you to "set kubeconfig file", "install the CNI" and "add the worker nodes using the created tokens". Below is a reference for you:

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join x.x.x.x:6443 --token gnl7u2.ibvu8iii9fjwvxxrj \
	--discovery-token-ca-cert-hash sha256:j8ud6000038456f2eb8ecejjja061e85380a312a704uyt1f27aeb6a0aabdoooo
  
# Seeting up Calico (RUN THIS ONLY ON YOUR MASTER NODE)
# Details about CNI https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
kubectl apply -f calico.yaml

kubectl get nodes

# Add the worker to the master node by running Join command on your Worker nodes

# Wait for few seconds and check the cluster status from your master by running:

kubectl get nodes
