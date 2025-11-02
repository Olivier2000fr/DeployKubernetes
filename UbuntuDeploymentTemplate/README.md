# 🚀 Automatisation du Déploiement d'Ubuntu LTS avec Cloud-Init

L'objectif de cette section est d'automatiser au maximum le déploiement des Machines Virtuelles **Ubuntu LTS** (Long Term Support) sur VMware Workstation Pro. Nous utilisons la fonctionnalité **cloud-init** (méthode **Autoinstall**) pour effectuer une installation sans interaction.

Le principe est de créer un **ISO d'amorçage** (`seed.iso`) contenant les fichiers de configuration nécessaires (`user-data` et `meta-data`). Cet ISO sera ensuite monté sur la VM pour déclencher l'installation automatique.

---

## 💻 Prérequis : Windows Subsystem for Linux (WSL)

L'environnement **Linux (WSL)** est recommandé pour générer le fichier ISO, car il permet d'utiliser facilement les outils Linux nécessaires (`cloud-image-utils`).

### 1. Installation de WSL

Si WSL n'est pas installé, veuillez suivre la documentation officielle de Microsoft :

> [Documentation Microsoft pour l'installation de WSL](https://learn.microsoft.com/fr-fr/windows/wsl/install)

---

## 🔑 Étape 2 : Préparation du Mot de Passe Haché

Pour des raisons de sécurité, le mot de passe de l'utilisateur initial dans le fichier `user-data` doit être fourni sous forme **hachée (SHA-512)**.

1.  Lancez votre distribution Linux WSL :

    ```bash
    wsl
    ```

2.  Générez le hachage pour votre mot de passe souhaité. Dans votre cas (mot de passe initial : `Coucou`), vous utiliseriez la commande suivante, en remplaçant par votre propre mot de passe si nécessaire :

    ```bash
    mkpasswd -m sha-512 VotreMotDePasse
    ```

    > 💡 **Rappel :** Copiez le résultat du hachage (une longue chaîne commençant par `$6$...`).

3.  Mettez ce hachage à jour dans la section du mot de passe de **chaque fichier `user-data`** de déploiement.

---

## ⚙️ Étape 3 : Configuration des Fichiers de Déploiement

Les fichiers `user-data` et `meta-data` contiennent toutes les instructions pour l'installation (utilisateur, mot de passe haché, configuration réseau, etc.).

> 📝 **Note :** J'ai mis en référence chaque fichier pour mon propre cas d'utilisation (réseau `10.33.62.xx`). Assurez-vous d'adapter l'adresse IP, le masque et la passerelle dans `user-data` à votre environnement.

---

## 💿 Étape 4 : Génération de l'ISO d'Autoinstallation

Une fois que les fichiers `user-data` et `meta-data` sont prêts et se trouvent dans le même répertoire, utilisez les commandes suivantes dans WSL pour créer l'ISO d'amorçage.

1.  Lancez WSL :

    ```bash
    wsl
    ```

2.  Installez les outils nécessaires si ce n'est pas déjà fait :

    ```bash
    sudo apt-get update
    sudo apt-get install cloud-image-utils
    ```

3.  Générez le fichier ISO, que nous nommerons **`seed.iso`** :

    ```bash
    cloud-localds seed.iso user-data meta-data
    ```

Le fichier **`seed.iso`** est prêt. Il doit être monté comme **second CD/DVD drive** dans votre machine virtuelle Workstation Pro, tandis que l'ISO d'installation d'Ubuntu doit être monté sur le premier drive.

---