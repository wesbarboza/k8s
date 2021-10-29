# Criando cluster kubernetes 1.22 com kubeadm

## Cenário:

1. K8s Control Panel
2. K8s Worker1   
3. K8s Worker 2

*ubuntu-20.04, 4GB ram, 2 CPU*

Install Packages *cluster*
Initialize the cluster
Install the Calico network add-on *Control*
Join the worker nodes to the cluster *Workers*


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
## Faça login no nó do plano de controle (Observação: as etapas a seguir devem ser executadas em todos os três nós.).

#### Crie um arquivo de configuração para o containerd:
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
