# Provision
```bash
vagrant up
```

```
vagrant status
```

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master1|kmaster1.localhost.com|192.168.99.101|CentOS 7|2G|2|
|Node1|knode1.localhost.com|192.168.99.111|CentOS 7|1G|1|
|Node2|knode2.localhost.com|192.168.99.111|CentOS 7|1G|1|


# On both Kmaster and Kworker

## Pre-requisites 
### Update /etc/hosts
```
cat >>/etc/hosts<<EOF
192.168.99.101 kmaster1.localhost.com kmaster1
192.168.99.111 knode1.localhost.com knode1
192.168.99.112 knode2.localhost.com knode2
EOF

```

### Docker repository to install docker.
```
yum install -y -q yum-utils device-mapper-persistent-data lvm2 > /dev/null 2>&1
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo > /dev/null 2>&1
yum update -y && yum install -y docker-ce-18.06.2.ce > /dev/null 2>&1

mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d


systemctl enable docker
systemctl start docker

```

### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

```

### Disable Firewall
```
systemctl disable firewalld
systemctl stop firewalld

```
### Disable swap
```
sed -i '/swap/d' /etc/fstab
swapoff -a

```

### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

```

# Kubernetes Setup
### Add yum repository
``` 
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

```

### Install Kubernetes
```
yum install -y kubeadm kubelet kubectl

```
### Enable and Start kubelet service
```
systemctl enable kubelet
systemctl start kubelet

```


# On kmaster

### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=192.168.99.101 --pod-network-cidr=10.244.0.0/16

```
