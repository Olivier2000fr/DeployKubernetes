# 🚀 Déploiement Automatisé d'un Cluster Kubernetes Minimal

**Objectif** : Ce projet vise à automatiser le déploiement d'un cluster Kubernetes minimal (1 nœud maître + 2 nœuds workers) destiné à un environnement de test fonctionnel, et à préparer l'environnement pour le déploiement de vos premiers conteneurs.

---

## 📋 Architecture Cible

### 1. Configuration Matérielle des VMs

Le déploiement utilise des Machines Virtuelles (VMs) basées sur Ubuntu Server 22.04 LTS.

| Rôle                     | Nombre de VMs | vCPU | RAM (Initial) | RAM (Post-Install) | OS Recommandé           |
| :----------------------- | :------------ | :--- | :------------ | :----------------- | :---------------------- |
| **Master/Control Plane** | 1             | 2    | 4 GB          | 4 GB               | Ubuntu Server 22.04 LTS |
| **Worker Node** | 2             | 2    | 4 GB          | 2 GB               | Ubuntu Server 22.04 LTS |

> **⚠️ Note Importante concernant la RAM :**
> Le processus d'installation d'Ubuntu Server 22.04 LTS peut planter avec seulement 2 Go de RAM. **Déployez initialement chaque VM avec 4 Go.** Une fois l'installation terminée et la VM éteinte, vous pourrez réduire la RAM des nœuds Workers à 2 Go.

### 2. Configuration Réseau (LAN NAT)

L'environnement utilise un réseau privé NAT configuré dans VMware, basé sur **VMnetX** (où X est votre ID réseau personnalisé, par exemple `VMnet1`).

| Paramètre | Valeur |
| :--- | :--- |
| **Réseau** | `10.33.62.0/24` |
| **Passerelle (Gateway)** | `10.33.62.2` |
| **Plage DHCP** | Active à partir de `10.33.62.128` |

| Rôle | Adresse IP Statique |
| :--- | :--- |
| **Master** | `10.33.62.10` |
| **Worker 1** | `10.33.62.11` |
| **Worker 2** | `10.33.62.12` |

---

## 🔧 Prérequis Techniques

1.  **Logiciel de Virtualisation :**
    * **VMware Workstation Pro** est recommandé.
2.  **Image ISO :**
    * **Ubuntu Server 22.04 LTS** (ou version ultérieure).
3.  **Fichiers de Déploiement Automatisé :**
    * Les fichiers `user-data` et `meta-data` pour l'Autoinstallation sont disponibles dans le répertoire : **`UbuntuDeploymentTemplate`**.
    * **Clé d'Automatisation :** L'utilisateur de connexion automatique est `k8sadmin` et le mot de passe est défini via un hachage SHA-512 dans le fichier `user-data`.

---

## ⚙️ Processus de Déploiement

Le déploiement se déroule en trois phases : la création des VMs, l'installation automatisée d'Ubuntu, et l'optimisation des OS invités.

### Étape 1 : Création et Amorçage des VMs

Créez les trois Machines Virtuelles avec la configuration suivante :

| Composant | Configuration | Note |
| :--- | :--- | :--- |
| **vCPU** | 2 | |
| **RAM** | 4 Go | Obligatoire pour l'installation |
| **Disque** | 25 Go (SCSI/NVMe) | |
| **CD/DVD Drive 1** | Pointez vers l'ISO **Ubuntu Server 22.04 LTS** | ISO du système d'exploitation |
| **CD/DVD Drive 2** | Pointez vers l'ISO **`seed.iso`** | Contient les fichiers `user-data` (voir `UbuntuDeploymentTemplate`) |

#### **Procédure d'Installation Automatisée :**

1.  **Démarrez** la première VM (Master).
2.  Dès l'invite **GRUB**, appuyez sur la touche `<Entrée>`.
3.  Le programme d'installation passera en mode graphique. Attendez quelques secondes. L'installeur **détectera automatiquement** les fichiers `user-data` sur le `seed.iso` et lancera l'installation en mode pré-rempli.
4.  Une fois l'installation du Master terminée, répétez le processus pour **Worker 1** et **Worker 2**.

### Étape 2 : Nettoyage et Optimisation des OS

Par défaut, l'installation Ubuntu peut activer des services graphiques et des outils inutiles dans un environnement serveur minimal, gaspillant des ressources.

Connectez-vous à chaque VM (Master, Worker 1, Worker 2) en utilisant votre client SSH favori (WSL, PuTTY, etc.) avec l'utilisateur **`k8sadmin`** et lancez les commandes de nettoyage :

```bash
# Se connecter avec le user 'k8sadmin' (mot de passe défini dans user-data)

# 1. Désactiver l'environnement graphique et ses dépendances
sudo systemctl set-default multi-user.target
sudo systemctl disable gdm3 

# 2. Désactiver les services inutiles
sudo systemctl disable snapd 
sudo systemctl disable bluetooth 
sudo systemctl disable cups 
sudo systemctl disable avahi-daemon

# 3. Nettoyer les dépendances et les caches
sudo apt autoremove --purge
sudo apt clean

# Mise à jour système
sudo apt update && sudo apt upgrade -y
sudo systemctl disable unattended-upgrades
# Installation des paquets essentiels uniquement
sudo apt install -y curl apt-transport-https ca-certificates gpg vim net-tools
# Désactivation du swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Modules kernel pour Kubernetes
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
# Paramètres sysctl
sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

```



Ne pas oublier Redimensionner la RAM (Applicable aux Workers uniquement)
Après avoir éteint les VMs, vous pouvez réduire la RAM des Workers à 2 Go.

rajout dans les fichier /etc/hosts des informations sur les IP et les neouds
```bash
10.33.62.10  k8s-master-01
10.33.62.11  k8s-worker-01
10.33.62.12  k8s-worker-02
```

### Étape 2 : Installation de Kubernetes

A faire sur toutes les VM's :
- installation de containerd
- installation de kubelet et kubeadm

```bash
# Installation de containerd
sudo apt install -y containerd
# Configuration de containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
# Activation de SystemdCgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Ajout du dépôt Kubernetes
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Installation
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


```


### Étape 3 : Initialisation du cluster 

Sur le Master node only
```bash
# Initialisation avec kubeadm
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
# Configuration de kubectl pour votre utilisateur
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# Installation d'un plugin réseau (Flannel)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
Important : Notez la commande kubeadm join qui s'affiche à la fin, vous en aurez besoin pour les workers !
Mon cas : 
`kubeadm join 10.33.62.10:6443 --token qpz0dd.3gansvvlbqkkm6rk \ `<br>
`        --discovery-token-ca-cert-hash sha256:076dc1889c76fab79b9eff1cdeec8d71f5eed9534e86c443d83cbdcabe80ae23`


Ensuite, il faut se connecter sur chaque worker et lancer la commande précédente en sudo.
`sudo kubeadm join <MASTER-IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>`

Maintenant on peut vérifier sur le master à l'aide des commandes suivantes : 

```bash
# Installation de containerd
kubectl get nodes
kubectl get pods -A
```

voici un exemple de retour : 

```
k8sadmin@k8s-master-01:~$ kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
k8s-master-01   Ready    control-plane   3m12s   v1.28.15
k8s-worker-01   Ready    <none>          55s     v1.28.15
k8s-worker-02   Ready    <none>          20s     v1.28.15
k8sadmin@k8s-master-01:~$ kubectl get pods -A
NAMESPACE      NAME                                    READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-t4vz4                   1/1     Running   0          62s
kube-flannel   kube-flannel-ds-w4lfj                   1/1     Running   0          27s
kube-flannel   kube-flannel-ds-xvbqt                   1/1     Running   0          118s
kube-system    coredns-5dd5756b68-2tcw9                1/1     Running   0          3m
kube-system    coredns-5dd5756b68-tzjvm                1/1     Running   0          3m
kube-system    etcd-k8s-master-01                      1/1     Running   0          3m16s
kube-system    kube-apiserver-k8s-master-01            1/1     Running   0          3m16s
kube-system    kube-controller-manager-k8s-master-01   1/1     Running   0          3m17s
kube-system    kube-proxy-2q9pb                        1/1     Running   0          3m
kube-system    kube-proxy-6m4r2                        1/1     Running   0          27s
kube-system    kube-proxy-w9xkl                        1/1     Running   0          62s
kube-system    kube-scheduler-k8s-master-01            1/1     Running   0          3m16s
k8sadmin@k8s-master-01:~$
```

### Étape 4 : Préparer son client Kube afin de ne pas utiliser les VM 

Nous allos utiliser les services Linux de Windows (WSL) directement pour passer des commandes Kube et ne pas utiliser de connexion aux VM's Déployées.

Pour cela, nous devons installer Kubeadm directement dans WSL et lui donner la configuraiton du master node

Il faut se connecter dans WSL et passer les commandes suivantes :

```bash
# installation de Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
# Déplacement vers /usr/local
sudo mv kubectl /usr/local/bin/
# Activer l'auto completion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
# Création du répertoire de configuration
mkdir -p ~/.kube
cd ~/.kube
```

Maintenant il va falloir aller récupérer le fichier de configuration sur le master node
```
olivier@DESKTOP-O96A07T:~/.kube$ sftp k8sadmin@10.33.62.10
k8sadmin@10.33.62.10's password:
Connected to 10.33.62.10.
sftp> cd .kube
sftp> ls
cache   config
sftp> get config
Fetching /home/k8sadmin/.kube/config to config
config                                                                                100% 5647     3.9MB/s   00:00
sftp> bye
```

et on peut vérifier que tout est bon !!!

```bash
kubectl get nodes
```
