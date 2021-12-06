# Configure DNS

```
systemd-resolve --interface enp0s3 --set-dns 8.8.8.8
```
NOTE: Only in Ubuntu 20.04

# Upgrade system

```
apt-get update && \
apt-get upgrade -y
```

# Configure system network

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

#  Install Docker

```
apt-get install -y docker.io
```

# Configure CGroup

```
sudo mkdir /etc/docker && \
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker && \
sudo systemctl daemon-reload && \
sudo systemctl restart docker
```

# Install kubeadm, kubelet and kubectl

```
sudo apt-get update && \
sudo apt-get install -y apt-transport-https ca-certificates curl && \
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg && \
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
sudo apt-get update && \
sudo apt-get install -y kubelet kubeadm kubectl && \
sudo apt-mark hold kubelet kubeadm kubectl
```

# Create cluster with kubeadm

```
sudo kubeadm init --pod-network-cidr=192.168.5.0/24 --apiserver-advertise-address=192.168.50.10
```

NOTE: In case we need to add more IPs to the certificate we should use the option --apiserver-cert-extra-sans [IPs spplited with comma]

# Configure the environment (kube/config)

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# Download CNI Calico definition

```
wget https://docs.projectcalico.org/manifests/tigera-operator.yaml && \
wget https://docs.projectcalico.org/manifests/custom-resources.yaml
```

# Modify the network CIDR in the Calico CNI definition

Edit the custom-resources.yaml file and set the correct CIDR (192.168.5.0/24)

# Apply the Calico CNI definition

```
kubectl create -f tigera-operator.yaml && \
kubectl create -f custom-resources.yaml && \
kubectl taint nodes --all node-role.kubernetes.io/master-
```

# Check everything is working with master node

```
kubectl get nodes -o wide
```

# List available tokens

```
kubeadm token list
```

# Create a new token

```
kubeadm token create
```

# Get the SHA hash

```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

# Join worker node to the cluster

```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

NOTE: control-plane-port is 6443

# Check everything is working with all nodes

```
kubectl get nodes -o wide
```

# Remove k8s

```
sudo kubeadm reset && \
sudo rm -fr /etc/cni && \
rm $HOME/.kube/config
```

# Remove all resources in docker

```
docker system prune -a
```
