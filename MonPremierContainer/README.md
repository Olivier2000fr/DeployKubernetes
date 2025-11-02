On va essayer de déployer et tester son premier container !!!

### Avant propos :

NGINX est un serveur web open source et un reverse proxy, connu pour sa légèreté, sa rapidité et sa capacité à gérer un grand nombre de connexions simultanées. Il est souvent utilisé pour servir des sites web statiques, équilibrer la charge entre plusieurs serveurs, ou comme proxy inverse pour des applications web (comme Node.js ou PHP). NGINX excelle aussi dans la gestion du cache et la sécurité (SSL/TLS). Il est largement adopté pour sa performance et sa faible consommation de ressources.

Il est très utilisé dans le mode container (un digne remplaçant d'Apache)

Pour déployer l'application Nginx vue précédemment, vous avez besoin de deux fichiers :
- Un manifeste pour le Deployment (qui gère les Pods).
- Un manifeste pour le Service (qui expose le Deployment).

### Manifeste du Déploiement (nginx-deployment.yaml)
Ce fichier va créer trois Pods Nginx à partir de l'image nginx:latest.

Explication des points clés :
- `kind: Deployment` : Spécifie le type d'objet Kubernetes à créer.
- `metadata.name: nginx-web` : Le nom que nous donnons à ce déploiement.
- `spec.replicas: 3` : Assure que Kubernetes maintient en permanence trois instances de cette application.
- `spec.selector.matchLabels` : Définit les étiquettes que ce Deployment doit gérer.
- `spec.template.metadata.labels` : Les étiquettes que les Pods créés par ce Deployment recevront. C'est le lien entre le Deployment et le Service.

### Manifeste du Service (nginx-service.yaml)
Ce fichier va exposer les Pods créés par le Deployment. Nous utilisons ici le type NodePort pour l'accès externe.

Explication des points clés :
- `kind: Service` : Spécifie le type d'objet Kubernetes à créer.
- `spec.selector.app: web` : Crucial. Il recherche et regroupe tous les Pods qui correspondent à l'étiquette app: web (ceux créés par notre Deployment Nginx).
- `ports.port: 80` : Le port par lequel le Service lui-même est accessible au sein du cluster.
- `ports.targetPort: 80` : Le port du conteneur réel vers lequel le trafic est transféré.
- `type: NodePort` : Permet l'accès depuis votre machine hôte via l'adresse IP d'un nœud et le port nodePort (30080).

### Déploiement des Manifestes
Enregistrez les deux contenus ci-dessus dans des fichiers distincts (nginx-deployment.yaml et nginx-service.yaml) sur votre nœud Master.

Vous pouvez les déployer en une seule commande en spécifiant le répertoire . (si les fichiers sont dans le répertoire courant) :

```bash
# Applique les deux manifestes au cluster
kubectl apply -f .
```


### Vérification
Vérifiez le statut de vos objets Kubernetes :

Bash

# Vérifie que le déploiement est en cours
kubectl get deployment nginx-web

# Vérifie que les 3 Pods sont en état 'Running'
kubectl get pods -l app=web

# Vérifie que le Service est créé et affiche le port NodePort
kubectl get service nginx-exposure
Vous pourrez ensuite accéder à votre application Nginx en utilisant l'IP de votre nœud et le port 30080 :

http://<IP_DU_WORKER>:30080