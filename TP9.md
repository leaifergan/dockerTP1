# Docker - TP9 : Utilisation de Docker-Compose
> **Objectifs du TP** :
>- Prise en main de l'outil docker-compose
>

### CE TP fait suite à ce que nous avons réalisé lors des TPs précédants

## 2- Création du docker-compose

Maintenant que nous avons créé notre fichier Dockerfile, il est temps de créer notre fichier docker-compose qui va nous simplifier la construction et le lancement de notre application. Nous allons créer un fichier `docker-compose.yml` (toujours dans votre repertoire de travail docker-app/`) avec le contenu suivant :
```yaml
# docker-app/docker-compose.yml
---
version: '3'
services:
  app:
    build: .
    image: "$UTILISATEUR/app:v0.1"
    ports:
    - "8000:8000"
```

Nous allons à présent utiliser docker-compose une première fois pour construire notre image :
```
dev $ docker-compose build
Building app
Step 1/11 : FROM python:3.7-alpine
 ---> c02a3409ee5b
Step 2/11 : LABEL maintainer="<Nom> <Prénom>"
 ---> Using cache
 ---> a2e9b278045e
Step 3/11 : WORKDIR /app
 ---> Running in 3ce50832d967
Removing intermediate container 3ce50832d967
 ---> d07cfb69c642
Step 4/11 : COPY requirements.txt .
 ---> fb52dbeb058f
Step 5/11 : RUN pip install -r requirements.txt
 ---> Running in e70df4a68b46
Collecting Flask==1.0.2 (from -r requirements.txt (line 2))
[]
Installing collected packages: MarkupSafe, Jinja2, itsdangerous, click, Werkzeug, Flask, gunicorn
Successfully installed Flask-1.0.2 Jinja2-2.10 MarkupSafe-1.1.0 Werkzeug-0.14.1 click-7.0 gunicorn-19.9.0 itsdangerous-1.1.0
Removing intermediate container e70df4a68b46
 ---> f1dac0f790ee
Step 6/11 : COPY app.py .
 ---> 597e9dfdca20
Step 7/11 : COPY config.py .
 ---> dc54fc18a0a7
Step 8/11 : RUN adduser -D my-user -u 1000
 ---> Running in 42050e09c8b6
Removing intermediate container 42050e09c8b6
 ---> f4c50ec0ccb9
Step 9/11 : USER 1000
 ---> Running in dc252f7b047c
Removing intermediate container dc252f7b047c
 ---> 06b4274f9bae
Step 10/11 : ENTRYPOINT ["gunicorn"]
 ---> Running in 6027d960cfdf
Removing intermediate container 6027d960cfdf
 ---> 39c5abb4193d
Step 11/11 : CMD ["--config", "config.py", "app:app"]
 ---> Running in b9260f53b0a6
Removing intermediate container b9260f53b0a6
 ---> 31920ffe5099
Successfully built 31920ffe5099
Successfully tagged $UTILISATEUR/app:v0.1
```
Cette commande utilise `docker-compose`, mais en réalité, elle ne fait qu’encapsulser pour vous un appel à la commande `docker image build` :
```sh
dev $ docker image build -t=$UTILISATEUR/app:v0.1 .
```
Cette commande ordonne à Docker de créer une nouvelle image en respectant les instructions contenues dans le Dockerfile que nous venons de créer. Nous affectons également un nom à notre nouvelle image afin de pouvoir l'identifier avec le flag -t. Nous pouvons optionnellement affecter un **tag** de version différent de `latest` en ajoutant “:” puis le nom de la version à la suite du nom de l’image (`v0.1` dans notre cas).

L'exécution des instructions avec la commande `RUN` suit un workflow très particulier :
1. Docker lance un conteneur à partir de l'image de base
2. Une instruction est exécutée, par exemple `pip install -r requirements.txt` et un changement est effectué dans le conteneur.
3. Docker déclenche l'équivalent d'un `docker container commit` afin de créer une nouvelle couche de fichiers. Puis il crée une nouvelle image à partir de cette couche par dessus les couches précédentes.
4. Docker lance un nouveau conteneur grâce à cette nouvelle image.
5. On répète jusqu'à ce que toutes les commandes `RUN` soient exécutées et que l'on soit en possession d'une image finale à utiliser.

On peut remarquer que l’ID de l’image construite nous est communiqué en fin de build : `31920ffe5099` ici.

Il est intéressant de noter qu'il existe une limitation importante à Docker (ou plutôt du système de fichiers dit union fs : Overlay2) : **Nous sommes limités à 127 couches de fichiers !**

Si nous créons un Dockerfile et que nous atteignons la limite de 127 couches, nous ne pouvons plus construire par-dessus. Bien qu'en pratique cette limite est difficilement atteignable, il est important de l'avoir en tête afin de factoriser au maximum les instructions dans notre Dockerfile.

Un avantage de la factorisation à lieu lors de la suppression des fichiers. En effet, comme dans Git, la suppression d’un fichier lors d’un commit ne le supprime pas de l’historique (la couche précédente dans notre cas). Il est donc recommandé de procéder aux tâches de purge dans le mêmes lignes que celle qui font des installations. Exemple :
```Dockerfile
RUN apt-get -y update && apt-get install -y \
  aufs-tools \
  automake \
  build-essential \
  curl \
  dpkg-sig \
  libcap-dev \
  libsqlite3-dev \
  mercurial \
  reprepo \
  ruby1.9.1 \
  ruby1.9.1-dev \
  s3cmd=1.1.* \
  && rm -rf /var/lib/apt/lists/*
```
Notez bien que toutes les commandes sont enchaînées dans une seule instruction `RUN` du `Dockerfile` via des `&&` et se termine par un `rm`. Une seule couche est créée et les fichiers temporaires sont supprimés avant le commit.

Il est à présent temps de démarrez l’application. Pour ce faire, exécutez la commande suivante :
```sh
dev $ docker-compose up
Creating network "docker-app_default" with the default driver
Creating docker-app_app_1 ... done
Attaching to docker-app_app_1
app_1 | [2019-01-08 15:53:33] [1] [INFO] Starting gunicorn 19.9.0
app_1 | [2019-01-08 15:53:33] [1] [INFO] Listening at: http://0.0.0.0:8000 (1)
app_1 | [2019-01-08 15:53:33] [1] [INFO] Using worker: gthread
app_1 | [2019-01-08 15:53:33] [7] [INFO] Booting worker with pid: 7
```

Ouvrez un autre terminal et essayez de requêter l’application :
```sh
dev $ curl localhost:8000 -v
...
>
< HTTP/1.1 200 OK
< Server: gunicorn/19.9.0
< Date: Fri, 06 Jul 2018 15:27:33 GMT
< Connection: close
< Content-Type: text/html; charset=utf-8
< Content-Length: 12
<
* Closing connection 0
Hello docker!
```
On voit que l’application répond avec un status code HTTP 200 et le message “Hello docker!”.

## 3- Nettoyage des conteneurs

Nous allons supprimer tous les conteneurs :
```sh
dev $ docker system prune
```
La commande `prune` nous permet de supprimer les ressources Docker qui ne sont plus utilisées dans notre machine. Dans ces ressources, on trouve les conteneurs arrêtés, les réseaux non-utilisés, les images sans nom (dangling), et les volumes.

## 4- Résumé des commandes

Bravo! Voici un petit résumé des commandes que nous avons vues dans cette section:

`docker image ls` Liste les images que l'on possède en local.
`docker search` Cherche une image dans le registre central (Docker Hub)
`docker image pull` Télécharge une image depuis le Docker Hub ou un registre privé.

`docker container commit` Sauvegarde les changements réalisés dans une image (installation d'un paquet, etc.) et crée une image réutilisable.

`Dockerfile` Fichier permettant de créer une image en décrivant une suite d'instructions.
- `FROM` L'image de base que l'on doit utiliser pour la construction de notre nouvelle image.
- `MAINTAINER` Le créateur/mainteneur de l'image.
- `RUN` Exécute une commande shell et crée une nouvelle couche de système de fichiers en fonctions des changements effectués.

`docker image build` Construit une image à partir d'un Dockerfile.
- `--tag / -t` Nom de l'image à construire.

`docker image history` Donne des informations sur la hiérarchie de couches de 	notre conteneur.

## 5- Question bonus

>**Question pour les plus rapides**
>- Dans les faits, nous n’utilisons jamais la commande `docker container commit`. Sauriez-vous dire pourquoi son usage n’est pas recommandé ?
