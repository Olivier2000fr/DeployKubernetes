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

Vous pouvez les déployer en une seule commande en spécifiant le répertoire. (si les fichiers sont dans le répertoire courant) :

```bash
# Applique les deux manifestes au cluster
kubectl apply -f .
```

On peut sinon : `kubectl apply -f ton-fichier-deployment.yaml`


### Vérification
Vérifiez le statut de vos objets Kubernetes :

```bash
# Vérifie que le déploiement est en cours
kubectl get deployment nginx-web
# Vérifie que les 3 Pods sont en état 'Running'
kubectl get pods -l app=web
```

Vérifiez que le Service est créé et affiche le port NodePort
```bash
kubectl get service nginx-exposure
```

Vous pourrez ensuite accéder à votre application Nginx en utilisant l'IP de votre nœud et le port 30080 :
```
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-exposure   NodePort   10.111.187.42   <none>        80:30080/TCP   30s
```
Donc http://10.33.62.11:30080 / http://10.33.62.12:30080 / http://10.33.62.10:30080 

Et cela marche aussi sur le master node et le 2ume worker node

### Destruction et nettoyage

Il faut lister les services qui tournent et détruire celui qui nous interesse :
```
olivier@DESKTOP-O96A07T:/mnt/c/Users/olivi/PycharmProjects/DeployKubernetes/MonPremierContainer$ kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        73m
nginx-exposure   NodePort    10.111.187.42   <none>        80:30080/TCP   10m
olivier@DESKTOP-O96A07T:/mnt/c/Users/olivi/PycharmProjects/DeployKubernetes/MonPremierContainer$ kubectl delete service nginx-exposure
```
Il faut supprimer le service puis le déploiement

```bash
kubectl delete service nginx-exposure
kubectl delete deployment nginx-web
```

### Jouer avec les logs

Les logs ne sont ni plus ni moins que l'output de la commande lancée par le container (dans notre cas nginx)

Il faut utiliser les commandes :
```bash
kubectl logs <nom-du-pod>
```

On peut rajouter quelques options:
- `--tail=10` Pour n'afficher que les 10 dernières lignes
- `-f` Pour etre en flux sur les écritures (et donc voir au fur et à mesure l'avancée des logs)

Exemple : 
```
kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-web-68888c5c95-4nzrz   1/1     Running   0          26s
nginx-web-68888c5c95-6ztft   1/1     Running   0          26s
nginx-web-68888c5c95-c59bz   1/1     Running   0          26s
```

```
kubectl logs nginx-web-68888c5c95-4nzrz
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/11/02 12:37:25 [notice] 1#1: using the "epoll" event method
2025/11/02 12:37:25 [notice] 1#1: nginx/1.29.3
2025/11/02 12:37:25 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2025/11/02 12:37:25 [notice] 1#1: OS: Linux 6.14.0-34-generic
2025/11/02 12:37:25 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/11/02 12:37:25 [notice] 1#1: start worker processes
2025/11/02 12:37:25 [notice] 1#1: start worker process 29
2025/11/02 12:37:25 [notice] 1#1: start worker process 30
```




