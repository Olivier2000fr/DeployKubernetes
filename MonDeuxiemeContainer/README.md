## Création de son propre container avec docker et publication de celui-ci

Cette partie est un peu plus compliquée !!!!

Le but va etre déployé un container contenant notre site wen préalablement réalisé par nos soins.

La Structure du site est comme suit :
- mon-site est mon site internet
- mon-site/index.html est ma page principale (et la seule du site)
- mon-site/Dockerfile est la définition de mon container.

Il faut donc au préalable avoir :
- installer docker sur son environnement de Développement
- Créer un compte sur le site Docker et avoir son propre repository

Il faut repérer son nom de repository dans son profile Docker. Pour cela, vous pouvez aller sur les settings de votre account et cela apparaitra dans l'URL : https://app.docker.com/accounts/guyoto/settings/account-information (donc mon cas c'est guyoto)

Ensuite il faut aller sur son PC et se connecter en ligne de commande à Docker avec le WSL.

la commande est :

```bash
docker login
```

Voici un exemple de login : 
```
docker login -u guyoto

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings


Password:

WARNING! Your credentials are stored unencrypted in '/home/olivier/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

J'ai visiblement des soucis de droits sur mon WSL donc j'ai du me relogger aussi en utilisant la commande 
```bash
sudo docker login
```

Il faut ensuite aller dans le répertoire mon-site et créer l'image docker :
```bash
sudo docker build -t mon-site-web:1.0 .
```

voici mon log : 
```
sudo docker build -t mon-site-web:1.0 .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  13.31kB
Step 1/3 : FROM nginx:alpine
alpine: Pulling from library/nginx
2d35ebdb57d9: Pulling fs layer
8f6a6833e95d: Pulling fs layer
194fa24e147d: Pulling fs layer
3eaba6cd10a3: Pulling fs layer
df413d6ebdc8: Pulling fs layer
d9a55dab5954: Pulling fs layer
ff8a36d5502a: Pulling fs layer
bdabb0d44271: Pulling fs layer
3eaba6cd10a3: Waiting
ff8a36d5502a: Waiting
bdabb0d44271: Waiting
d9a55dab5954: Waiting
df413d6ebdc8: Waiting
194fa24e147d: Verifying Checksum
194fa24e147d: Download complete
8f6a6833e95d: Verifying Checksum
8f6a6833e95d: Download complete
2d35ebdb57d9: Verifying Checksum
2d35ebdb57d9: Download complete
2d35ebdb57d9: Pull complete
8f6a6833e95d: Pull complete
3eaba6cd10a3: Verifying Checksum
3eaba6cd10a3: Download complete
194fa24e147d: Pull complete
3eaba6cd10a3: Pull complete
d9a55dab5954: Verifying Checksum
d9a55dab5954: Download complete
df413d6ebdc8: Download complete
df413d6ebdc8: Pull complete
d9a55dab5954: Pull complete
ff8a36d5502a: Verifying Checksum
ff8a36d5502a: Download complete
ff8a36d5502a: Pull complete
bdabb0d44271: Download complete
bdabb0d44271: Pull complete
Digest: sha256:b3c656d55d7ad751196f21b7fd2e8d4da9cb430e32f646adcf92441b72f82b14
Status: Downloaded newer image for nginx:alpine
 ---> d4918ca78576
Step 2/3 : COPY . /usr/share/nginx/html
 ---> 05535958d31a
Step 3/3 : EXPOSE 80
 ---> Running in ab0334c39171
 ---> Removed intermediate container ab0334c39171
 ---> dc497f57db1b
Successfully built dc497f57db1b
Successfully tagged mon-site-web:1.0
```
Il faut ensuite tagger le cotainer avec son nom de compte chez docker, puis le pousser sur Internet :

```bash
sudo docker tag mon-site-web:1.0 guyoto/mon-site-web:1.0
# lister les images docker disponibles
sudo docker images
# Pousser l'image citée vers le web
sudo docker push guyoto/mon-site-web:1.0
```

Voici mon log : 

```
sudo docker tag mon-site-web:1.0 guyoto/mon-site-web:1.0
sudo docker images
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
guyoto/mon-site-web    1.0       dc497f57db1b   25 minutes ago   52.9MB
mon-site-web           1.0       dc497f57db1b   25 minutes ago   52.9MB
olivier/mon-site-web   1.0       dc497f57db1b   25 minutes ago   52.9MB
nginx                  alpine    d4918ca78576   4 days ago       52.8MB
sudo docker push guyoto/mon-site-web:1.0
The push refers to repository [docker.io/guyoto/mon-site-web]
1a70328fc963: Pushed
25906c27b84d: Pushed
99ea4bde418d: Pushed
3297b9628ff3: Pushed
b74d92be8225: Pushed
2c79d5d895bb: Pushed
2660a7d4b906: Pushed
50b58ca2a3f5: Pushed
256f393e029f: Pushed
1.0: digest: sha256:c4cc8fd3fa84e865417057b22e89967e815ed3f816eb41d4a696a6c6a6aae93e size: 2197
```

On  peut ensuite Déployer container sur notre cluster avec
```bash
kubectl apply -f deployment.yaml
```

et ensuite, on peut manipuler le container comme vu dans le premier exemple.

## Essayons de comprendre comment fonctionne une image Docker

Voici son contenu : 
```bash
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```

`FROM nginx:alpine`
- Rôle : Définit l’image de base utilisée pour construire ton conteneur.
- :alpine : Spécifie une version légère de cette image, basée sur la distribution Linux Alpine. Cela réduit la taille du conteneur et améliore sa sécurité.


`COPY . /usr/share/nginx/html`
- Rôle : Copie les fichiers de ton dossier local (.) vers un dossier spécifique dans le conteneur.
- Détails :
  - . : Représente le dossier courant (où se trouve ton Dockerfile et les fichiers de ton site, comme index.html). 
  - /usr/share/nginx/html : Dossier par défaut où Nginx cherche les fichiers à servir (racine du site web).


`EXPOSE 80`
- Rôle : Documente le port sur lequel le conteneur écoutera.
- Détails :
  - 80 : Port standard pour le trafic HTTP.
  - Attention : EXPOSE est une déclaration informative pour les utilisateurs du conteneur. Pour que le port soit effectivement accessible depuis l’extérieur, il faut aussi utiliser l’option -p lors du docker run (ex: -p 8080:80).

Si à chaque version de code HTML nous sommes obligés de recréer un container, nous ne sommes pas sortis des ronces !!!!

