# Déploiement Automatisé d'un Cluster Kubernetes

**Objectif** : Automatiser le déploiement d'un cluster Kubernetes minimal (1 nœud maître + 2 nœuds workers) pour un environnement de test fonctionnel et déployer ses premiers container.


---

## 📋 Architecture Recommandée

### Configuration Matérielle
   Rôle               | Nombre de VMs | vCPU | RAM   | OS Recommandé          |
 |--------------------|---------------|------|-------|------------------------|
 | **Master/Control Plane** | 1             | 2    | 2-4 GB | Ubuntu Server 22.04 LTS |
 | **Worker Nodes**       | 2-3           | 2    | 2 GB   | Ubuntu Server 22.04 LTS |

### Environnement
- **Virtualisation** : VMware Workstation (ou équivalent comme VirtualBox).
- **Système d'exploitation** : Ubuntu Server 22.04 LTS (léger, stable et bien documenté).

---

## 🚀 Pourquoi ce projet ?
- **Automatisation** : Réduire les étapes manuelles pour un déploiement rapide et reproductible.
- **Test fonctionnel** : Idéal pour expérimenter Kubernetes sans ressources lourdes.
- **Documentation** : Faciliter la prise en main pour les débutants et les équipes.

## 🔧 Prérequis

### Choses à préparer
1. **Installer VMware Workstation Pro** ([Lien de téléchargement](https://www.vmware.com/products/workstation-pro.html)).
2. **Télécharger une version LTS d'Ubuntu Server** (ex : Ubuntu Server 22.04 LTS) ([Lien de téléchargement](https://ubuntu.com/download/server)).
3. **Configurer l'environnement réseau** :
   - Utiliser un **LAN NAT** sous VMware avec les paramètres suivants :
     - **Réseau** : `10.33.62.0/24` *(Note : Le masque `/255` a été corrigé en `/24` pour une notation CIDR standard.)*
     - **Passerelle** : `10.33.62.2`
     - **Plage DHCP** : Active à partir de `10.33.62.128` *(Note : Corrigé pour correspondre au réseau `10.33.62.0/24`.)*
   - **Adresses IP fixes pour les VMs** :
     - **Master** : `10.33.62.10`
     - **Worker 1** : `10.33.62.11`
     - **Worker 2** : `10.33.62.12` *(Note : Corrigé pour correspondre au réseau `10.33.62.0/24`.)*
