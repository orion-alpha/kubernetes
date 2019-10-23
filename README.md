##Создание кластера Kubernetes 

###Подготовка

Останавливаем файрвол
```
systemctl stop firewalld
systemctl disable firewalld
```

или добавляем разрешающие правила

```
Kubernestes needs:

Master node(s):
TCP     6443*       	Kubernetes API Server
TCP     2379-2380   	etcd server client API
TCP     10250       	Kubelet API
TCP     10251       	kube-scheduler
TCP     10252       	kube-controller-manager
TCP     10255       	Read-Only Kubelet API

Worker nodes (minions):
TCP     10250       	Kubelet API
TCP     10255       	Read-Only Kubelet API
TCP     30000-32767 	NodePort Services
```
Добавляем репозиторий:
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
Переключаем SELinux в permissive mode

```
setenforce 0

sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

Для того, чтобы можно было с помощью iptables фильтровать трафик, проходящий транзитом через мост необходимо выставить следующие системные переменные sysctl:
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
***
Не забываем предварительно установить среду запуска контейнеров ([container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/))

***

Oтключаем Swap и устанавливаем необходимые пакеты
```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
Запускаем службу
```
systemctl enable --now kubelet
```
Для организации сети буду использовать [flannel](https://github.com/coreos/flannel)

Скачиваем yml отвечающий за настройку Flannel
```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Cмотрим адрес сети
```
grep -i network kube-flannel.yml
```
(понадобится во время инициализации кластера) 

С помощью команды запускаем инициализацию управляющего узла, указываем адрес POD-сети кластера по умолчанию

```
kubeadm init --pod-network-cidr 10.244.0.0/16
```

