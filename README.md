# odin-server

Install k0s 1.29 on Fedora Server 39

## Prereqs

```bash
sudo hostnamectl hostname odin.anyops.site
sudo modprobe br_netfilter
sudo echo '1' > /proc/sys/net/ipv4/ip_forward
sudo dnf remove -y zram-generator # Disable swap
sudo swapoff --all
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
exclude=kubectl
EOF

sudo dnf install -y kubectl containerd helm --disableexcludes=kubernetes

sudo systemctl enable --now containerd
```

## Configure k0s

```bash
mkdir -p /etc/k0s
k0s config create > /etc/k0s/k0s.yaml

# Edit /etc/k0s/k0s.yaml and add any custom hostname
sudo k0s install controller --single -c /etc/k0s/k0s.yaml
```

## Install Cilium

```bash
helm repo add cilium https://helm.cilium.io/

helm install cilium cilium/cilium --version 1.15.2 --namespace kube-system

CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

cilium status --wait
```
