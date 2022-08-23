## Part-1 Sanal Makinaları Kubernetes Enviroment için Kuralım

- In this hands-on, we will prepare two nodes for Kubernetes on `Ubuntu 20.04`. One of the node will be configured as the Master node, the other will be the worker node. Following steps should be executed on all nodes. *Note: It is recommended to install Kubernetes on machines with `2 CPU Core` and `2GB RAM` at minimum to get it working efficiently. For this reason, we will select `t2.medium` as EC2 instance type, which has `2 CPU Core` and `4 GB RAM`.*

- Bu iki makine için gerekli Security Groups lar:

### Control-plane (Master) Node(s)

|Protocol|Direction|Port Range|Purpose|Used By|
|---|---|---|---|---|
|TCP|Inbound|6443|Kubernetes API server|All|
|TCP|Inbound|2379-2380|`etcd` server client API|kube-apiserver, etcd|
|TCP|Inbound|10250|Kubelet API|Self, Control plane|
|TCP|Inbound|10259|kube-scheduler|Self|
|TCP|Inbound|10257|kube-controller-manager|Self|
|TCP|Inbound|22|remote access with ssh|Self|
|UDP|Inbound|8472|Cluster-Wide Network Comm. - Flannel VXLAN|Self|

### Worker Node(s)

|Protocol|Direction|Port Range|Purpose|Used By|
|---|---|---|---|---|
|TCP|Inbound|10250|Kubelet API|Self, Control plane|
|TCP|Inbound|30000-32767|NodePort Services†|All|
|TCP|Inbound|22|remote access with ssh|Self|
|UDP|Inbound|8472|Cluster-Wide Network Comm. - Flannel VXLAN|Self|

- Hostname change of the nodes, so we can discern the roles of each nodes. For example, you can name the nodes (instances) like `kube-master, kube-worker-1`

```bash
sudo hostnamectl set-hostname <node-name-master-or-worker>
bash
```

- Kubernetes için yardımcı paketleri indirelim

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```

- Uygulama repository güncelleyelim ve Docker ve Kubernetes paketlerini indirelim

```bash
sudo apt-get update

sudo apt-get install -y kubectl kubeadm kubelet kubernetes-cni docker.io
```

- Docker service başlatalım ve enable edelim

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
- Docker'da `sudo` yazmadan komut yazmak için

```bash
sudo usermod -aG docker $USER
newgrp docker
```

- As a requirement, update the `iptables` of Linux Nodes to enable them to see bridged traffic correctly. Thus, you should ensure `net.bridge.bridge-nf-call-iptables` is set to `1` in your `sysctl` config and activate `iptables` immediately.

```bash
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## Part-2 Kubernetes için Master node nu kuralım

- Pull the packages for Kubernetes beforehand

```bash
sudo kubeadm config images pull
```

- By default, the Kubernetes cgroup driver is set to system, but docker is set to systemd. We need to change the Docker cgroup driver by creating a configuration file `/etc/docker/daemon.json` and adding the following line then restart deamon, docker and kubelet:

```bash
echo '{"exec-opts": ["native.cgroupdriver=systemd"]}' | sudo tee /etc/docker/daemon.json
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

- Senin için `kubeadm` enviromentı oluştursun. Note: master makinanın ip addresini alamayı unutma

```bash
sudo kubeadm init --apiserver-advertise-address=<ec2-private-ip> --pod-network-cidr=10.244.0.0/16
```
- Yukarıdaki komutu çalıştırdıktan sonra çıkan outputun ensonundaki kubeadm ile başlayan kısmı test.txt dosyası oluştur ve içine kopyala

- Yukarıdaki komutu çalıştırdıktan sonra;

```bash
-mkdir -p $HOME/.kube
-sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
-sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Worker'ı Master'a bağlamadan önce Worker'a bu komutları koştur

```bash
echo '{"exec-opts": ["native.cgroupdriver=systemd"]}' | sudo tee /etc/docker/daemon.json
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

- Sonra Worker node'nu bağlamak için test.txt ye kaydedilen komutu `sudo` ile çalıştır Worker'da

- Worker'in bağlandığını görmek için Master'da `kubectl get nodes` komutunu koştur

- Network'ü ayarlamak için Master'da

```bash
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```
- Yukarıdaki komutu girdikten sonra Mastera `kubectl get nodes` girdiğinde Status kısmı Ready oluyor.
