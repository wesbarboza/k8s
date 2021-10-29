## Criando cluster kubernetes 1.22 com kubeadm

Cenário:

K8s Control Panel

K8s Worker1   K8s Worker 2

Tree servers
one control plane and two worker nodes

Control:
```shell
cloud_user@k8s-control:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           388M  1.4M  387M   1% /run
/dev/nvme0n1p1   20G  4.2G   16G  22% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/loop0       56M   56M     0 100% /snap/core18/1988
/dev/loop1       34M   34M     0 100% /snap/amazon-ssm-agent/3552
/dev/loop3       99M   99M     0 100% /snap/core/10823
/dev/loop5       56M   56M     0 100% /snap/core18/2246
/dev/loop6      100M  100M     0 100% /snap/core/11993
/dev/loop4       25M   25M     0 100% /snap/amazon-ssm-agent/4046
tmpfs           388M     0  388M   0% /run/user/1001
shm              64M     0   64M   0% /run/containerd/io.containerd.grpc.v1.cri/sandboxes/d9a637db538f37e6b9c87a1d2b56b0fcefa7670145ca91e32c134458dce73494/shm
shm              64M     0   64M   0% /run/containerd/io.containerd.grpc.v1.cri/sandboxes/9edad6bfbce82b24d59044e142c90cd968be693ab74cdb859f4027f798dc04a0/shm
shm              64M     0   64M   0% /run/containerd/io.containerd.grpc.v1.cri/sandboxes/f547f578f2aa6d7bf19d6777dfd992d3c72e282935e1ced641e4b88baf1c3fec/shm
shm              64M     0   64M   0% /run/containerd/io.containerd.grpc.v1.cri/sandboxes/dade6fea7e0db2f7514625a5192a962652b9c0c6344e4dde9033fdf5a2913794/shm
shm              64M     0   64M   0% /run/containerd/io.containerd.grpc.v1.cri/sandboxes/4dffc9c9fe3372f89855e1cda62367f1b2da3fc69f42849fa0848cf3
```
2 processadores 
4 GB RAM

Worker
cloud_user@k8s-worker1:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           388M  760K  387M   1% /run
/dev/nvme0n1p1   20G  3.2G   17G  17% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/loop0       34M   34M     0 100% /snap/amazon-ssm-agent/3552
/dev/loop1       99M   99M     0 100% /snap/core/10823
/dev/loop2       56M   56M     0 100% /snap/core18/1988
/dev/loop5       56M   56M     0 100% /snap/core18/2246
/dev/loop6      100M  100M     0 100% /snap/core/11993
/dev/loop3       25M   25M     0 100% /snap/amazon-ssm-agent/4046
tmpfs           388M     0  388M   0% /run/user/1001

2 processadores
4GB RAM

q


Install Packages
Initialize the cluster
Install the Calico network add-on
Join the worker nodes to the cluster


## 1

Pacotes de instalação
Faça login em todos os três servidores.
Obtenha o containerd instalado e em execução.
Instale os pacotes do Kubernetes (kubeadm, kubelet e kubectl).


Inicialize o cluster
Inicialize o cluster Kubernetes no nó do plano de controle usando kubeadm.


Instale o complemento Calico Network
Instale o complemento de rede Calico em seu cluster.

Junte os nós de trabalho ao cluster
Junte os dois servidores de nó de trabalho ao cluster.
Use kubectl get nodesno nó do plano de controle para verificar se todos os três nós estão registrados com êxito e no READYestado.


---------
Pacotes de instalação
# Faça login no nó do plano de controle (Observação: as etapas a seguir devem ser executadas em todos os três nós.).

# Crie um arquivo de configuração para o containerd:
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF


# Módulos de carga:
sudo modprobe overlay
sudo modprobe br_netfilter

# Defina as configurações do sistema para a rede Kubernetes:
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Aplicar novas configurações:
sudo sysctl --system

# Instale o containerd:
sudo apt-get update && sudo apt-get install -y containerd

# Crie o arquivo de configuração padrão para containerd:
sudo mkdir -p /etc/containerd

# Gere a configuração padrão do containerd e salve no arquivo padrão recém-criado:
sudo containerd config default | sudo tee /etc/containerd/config.toml

# Reinicie o containerd para garantir o uso do novo arquivo de configuração:
sudo systemctl restart containerd

# Verifique se o containerd está em execução.
sudo systemctl status containerd

# Desativar troca:
sudo swapoff -a

# Desative a troca na inicialização em /etc/fstab:
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Instale pacotes de dependência:
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

# Baixe e adicione a chave GPG:
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
# Adicione o Kubernetes à lista de repositórios:

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

# Atualizar listas de pacotes:
sudo apt-get update

# Instale os pacotes do Kubernetes (Observação: se você receber uma mensagem de bloqueio do dpkg, espere um ou dois minutos antes de tentar o comando novamente):
sudo apt-get install -y kubelet=1.22.0-00 kubeadm=1.22.0-00 kubectl=1.22.0-00

# Desative as atualizações automáticas:
sudo apt-mark hold kubelet kubeadm kubectl
# Faça login em ambos os nós de trabalho para executar as etapas anteriores.

# Inicialize o cluster
# Inicialize o cluster Kubernetes no nó do plano de controle usando kubeadm (Observação: isso só é executado no nó do plano de controle) :
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.22.0

# Defina o acesso ao kubectl:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Teste o acesso ao cluster:
kubectl get nodes

# Instale o complemento Calico Network
# No nó do plano de controle, instale Calico Networking:
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Verifique o status do nó do plano de controle:
kubectl get nodes

# Junte os nós de trabalho ao cluster
# No nó do plano de controle, crie o token e copie o comando kubeadm join (OBSERVAÇÃO: o comando join também pode ser encontrado na saída do kubeadm initcomando) :
kubeadm token create --print-join-command

# Em ambos os nós de trabalho, cole o comando kubeadm join para ingressar no cluster. Use sudo para executá-lo como root:
sudo kubeadm join ...

# No nó do plano de controle, visualize o status do cluster (Observação: pode ser necessário aguardar alguns instantes para permitir que todos os nós estejam prontos):
kubectl get nodes
Conclusão
