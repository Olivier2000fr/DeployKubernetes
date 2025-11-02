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

# 4. Redimensionner la RAM (Applicable aux Workers uniquement)
# Après avoir éteint les VMs, vous pouvez réduire la RAM des Workers à 2 Go.