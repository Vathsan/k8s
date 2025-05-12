# k8s
Open relevant ports,
https://kubernetes.io/docs/reference/networking/ports-and-protocols/
 
1. On all nodes, set up Docker Engine and containerd. You will need to load some kernel modules and modify some system settings as part of this process

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

2. sysctl params required by setup, params persist across reboots
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```


3. Apply sysctl params without reboot

`sudo sysctl --system`

4. Set up the Docker Engine repository

`sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg lsb-release apt-transport-https`

5. Add Docker’s official GPG key
```
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

6. Set up the repository
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

7. Update the apt package index

`sudo apt-get update`

8. Install Docker Engine, containerd, and Docker Compose
```
VERSION_STRING=5:23.0.1-1~ubuntu.20.04~focal
sudo apt-get install -y docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

9. Add your 'cloud_user' to the docker group

`sudo usermod -aG docker $USER`

10. Log out and log back in so that your group membership is re-evaluated

11. Make sure that 'disabled_plugins' is commented out in your config.toml file

`sudo sed -i 's/disabled_plugins/#disabled_plugins/' /etc/containerd/config.toml`

12. Restart containerd

`sudo systemctl restart containerd`

13. On all nodes, disable swap.
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

14. On all nodes, install kubeadm, kubelet, and kubectl
```

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

15. On the control plane node only, initialize the cluster and set up kubectl access
```
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.29.1

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

16. Verify the cluster is working

`kubectl get nodes`

17. Install the Calico network add-on

`kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml`

18. Get the join command (this command is also printed during kubeadm init . Feel free to simply copy it from there)

`kubeadm token create --print-join-command`

19. Copy the join command from the control plane node. Run it on each worker node as root (i.e. with sudo )

`sudo kubeadm join ...`

20. On the control plane node, verify all nodes in your cluster are ready. Note that it may take a few moments for all of the nodes to enter the READY state

`kubectl get nodes`
