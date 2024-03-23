# odin-server

Install kubeadm 1.29 on Fedora Linux 39

## Prereqs

```bash
sudo hostnamectl hostname odin.anyops.site
sudo modprobe br_netfilter
sudo echo '1' > /proc/sys/net/ipv4/ip_forward
sudo dnf remove -y zram-generator # Disable swap
sudo swapoff --all
```

## Disable suspend on AC power

```bash
sudo -u gdm dbus-run-session gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
```

## Open ports

```bash
sudo firewall-cmd --zone=home --permanent --add-port=6443/tcp
sudo firewall-cmd --zone=home --permanent --add-port=10250/tcp
firewall-cmd --reload
```

## SELinux

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

## Install CNCF tools

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

sudo dnf install -y kubelet kubeadm kubectl containerd helm --disableexcludes=kubernetes

sudo systemctl enable --now containerd
sudo systemctl enable --now kubelet
```

## Configure kubeadm

```bash
kubeadm init --control-plane-endpoint=odin.anyops.site --node-name=odin.anyops.site --skip-phases=addon/kube-proxy
```

## Install Cilium

```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.15.2 --namespace kube-system
```
