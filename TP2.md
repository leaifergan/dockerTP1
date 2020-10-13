# Docker - TP2 : Utilisation des images
> **Objectifs du TP** :
>- Comprendre comment utiliser les images sur Docker
>

## 1- Les Images Docker et le Registre distant (Docker Hub)

Nous avons vu dans la première partie que pour lancer un conteneur, nous avons **besoin** d'une image. Il s’agit d’un modèle qui nous permet de créer notre conteneur, en d'autres termes tout ce qu'il contiendra en termes de fichiers et de programmes lors de son lancement. Dans nos exemples, nous avons créé des conteneurs utilisant une image de base Ubuntu.

Comme nous l'avons vu dans le cours, une image est composée de plusieurs couches en lecture seule. Cela permet de créer une hiérarchie réutilisable par tous les conteneurs. Si vous lancez deux conteneurs avec pour image de base Ubuntu, vous n'aurez pas deux fois 400 Mo de fichiers utilisés (comme pour des machines virtuelles), mais bien une fois 400 Mo.

Docker utilise overlay2 ou d'autres systèmes de fichiers comme overlay, BTRFS (comme on peut le voir avec la commande `docker info`) comme système de fichiers. C'est un système de fichiers unifié (_union mount_), qui permet de fusionner plusieurs systèmes de fichiers distincts et de les faire apparaître comme un unique système de fichiers avec une hiérarchie unifiée.

Ainsi plusieurs conteneurs peuvent monter leur système de fichiers avec des briques leur appartenant et d'autres partagées entre tous les conteneurs. Si un de ces conteneurs écrit dans un fichier, il va créer une copie qui lui est propre afin de pouvoir l’utiliser localement après cette modification (mécanisme de _copy on write_).

Les images sont des éléments réutilisables et il en existe pour beaucoup de variétés d'environnements (Java, Python, MySQL, Oracle DB, Postgres, Node.js, etc.). Afin de partager ces environnements, on utilise un **registre** (registry dans le vocabulaire Docker) dans lequel on va stocker ces modèles (et toute la structure de fichiers associée). Imaginez ce registre distant comme un ensemble de dépôts Subversion ou Git. De la même manière qu'on stocke du code, on stocke ici des briques de base constituant nos environnements afin de les réutiliser

## 2- Lister les images docker

Commençons par lister les images disponibles pour notre démon Docker. Les images listées sont celles qui sont directement disponibles localement afin de construire et lancer un conteneur.
```sh
dev $ docker image ls
REPOSITORY  TAG     IMAGE ID      CREATED      SIZE
ubuntu      latest  f753707788c5  4 weeks ago  127.2 MB
```
Les images qui ne sont pas listées doivent être téléchargées à partir du Docker Hub, qui est le registre hébergeant toutes les images de base mises à disposition par Docker et la communauté. N'importe qui peut créer un compte sur le Docker Hub et y stocker des images.

Les images sont stockées dans des dépôts (à la manière d'un repository SVN ou Git) qui sont eux-mêmes stockés dans un registre.
## 3- Télécharger une image Docker

Les images listées par la commande `docker images` ne font pas état d'une image alpine sur notre système. Tentons donc de la télécharger à partir du Docker Hub. Pour ça, on utilise la commande `docker image pull`. Vous devriez observer la sortie suivante :
```sh
dev $ docker image pull --all-tags alpine
2.6: Pulling from library/alpine
2a3ebcb7fbcc: Pull complete
Digest: sha256:e9cec9aec697d8b9d450edd32860ecd363f2f3174c8338beb5f809422d182c63
2.7: Pulling from library/alpine
4dea34575ff3: Pull complete
Digest: sha256:9f08005dff552038f0ad2f46b8e65ff3d25641747d3912e3ea8da6785046561a
3.1: Pulling from library/alpine
35a9f57fd9f2: Pull complete
Digest: sha256:2d74cbc2fbe3d261fdcca45d493ce1e3f3efd270114a62e383a8e45caeb48788
3.2: Pulling from library/alpine
e052f352ed4b: Pull complete
Digest: sha256:8565a58be8238ef688dbd90e43ec8e080114f1e1db846399116543eb8ef7d7b7
3.3: Pulling from library/alpine
c19324d1d971: Pull complete
Digest: sha256:06fa785d55c35050241c60274e24ad57025683d5e939b3a31cc94193ca24740b
3.4: Pulling from library/alpine
49388a8c9c86: Pull complete
Digest: sha256:915b0ffca1d76ac57d83f28d568bcb516b6c274843ea8df7fac4b247440f796b
3.5: Pulling from library/alpine
b1f00a6a160c: Pull complete
Digest: sha256:b007a354427e1880de9cdba533e8e57382b7f2853a68a478a17d447b302c219c
3.6: Pulling from library/alpine
b56ae66c2937: Pull complete
Digest: sha256:d6bfc3baf615dc9618209a8d607ba2a8103d9c8a405b3bd8741d88b4bef36478
edge: Pulling from library/alpine
e00d546a75ad: Pull complete
Digest: sha256:23e7d843e63a3eee29b6b8cfcd10e23dd1ef28f47251a985606a31040bf8e096
latest: Pulling from library/alpine
Digest: sha256:d6bfc3baf615dc9618209a8d607ba2a8103d9c8a405b3bd8741d88b4bef36478
Status: Downloaded newer image for alpine
```
Docker télécharge plusieurs couches pour l'image de base `alpine`.

Maintenant, découvrons les images téléchargées en relançant la commande `docker image ls`.
```sh
dev $ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              3.5                 6c6084ed97e5        5 months ago        3.99MB
alpine              3.4                 c7fc7faf8c28        5 months ago        4.82MB
alpine              3.3                 fc8815064a1b        5 months ago        4.81MB
alpine              3.2                 220b1d97bf6a        5 months ago        5.27MB
alpine              3.1                 2ba97bb89407        5 months ago        5.05MB
alpine              edge                5c4fa780951b        5 months ago        4.15MB
alpine              3.7                 3fd9065eaf02        5 months ago        4.15MB
alpine              latest              3fd9065eaf02        5 months ago        4.15MB
alpine              3.6                 77144d8c6bdc        5 months ago        3.97MB
alpine              2.7                 93f518ec2c41        2 years ago         4.71MB
alpine              2.6                 e738dfbe7a10        2 years ago         4.5MB
...
```
La commande liste non pas une mais dix versions (images) de la distribution `alpine` à partir de son dépôt stocké sur le [Docker Hub](https://hub.docker.com/_/alpine?tab=tags). On remarque que certaines versions correspondent bien aux couches que l'on a téléchargé précédemment grâce à la commande `docker image pull`. On peut donc choisir une image parmi toutes ces versions afin de démarrer un conteneur.

Lançons un conteneur avec une image de base `alpine`.
```sh
dev $ docker container run -i -t alpine /bin/sh
```
On obtient ainsi un conteneur avec un système de fichiers et des programmes correspondant à la dernière distribution alpine. Depuis l’intérieur du conteneur, vérifions quelle version de alpine a été démarrée avec la commande `cat /etc/os-release`.
```
# cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.8.0
PRETTY_NAME="Alpine Linux v3.8"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
```
Sortez maintenant de ce conteneur en tapant la commande `exit`.

Si on jette de nouveau un coup d'œil sur la sortie de la commande `docker image ls`, on remarque que deux images possèdent le même identifiant ! Comment cela peut-il être possible ? Une image n'est-elle pas censée posséder un identifiant unique ?
```sh
dev $ docker image ls
REPOSITORY                                               TAG       IMAGE ID
[...]
alpine                                                   3.5       a2b04ae28915
alpine                                                   3.4       993b1b41569d
alpine                                                   3.3       19bf2ec565c3
alpine                                                   3.2       e45221eb7f87
alpine                                                   3.1       6f8a01c2945d
alpine                                                   edge      9d1f27787d39
alpine                                                   3.8       11cd0b38bc3c
alpine                                                   latest    11cd0b38bc3c
alpine                                                   3.7       791c3e2ebfcb
alpine                                                   3.6       da579b235e92
[...]
```
En réalité, Docker utilise également des tags pour qualifier les images. Dans notre cas, les images avec les tags `3.8` et `latest` sont en réalité la **même image**. Par défaut, si on met juste `alpine`, Docker construit un nouveau conteneur avec la version marquée par un tag `latest`.

Pour choisir une autre distribution de alpine, par exemple `3.5`, il faut également spécifier le tag de l'image pour la commande docker container run.
```sh
dev $ docker container run -i -t alpine:3.5 /bin/sh
```
Encore une fois, depuis l’intérieur du conteneur, vérifions quelle version de alpine a été démarrée avec la commande `cat /etc/os-release`.
```
# cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.5.2
PRETTY_NAME="Alpine Linux v3.5"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
```
Et voilà ! Nous avons cette fois-ci crée un conteneur avec comme image de base une distribution alpine 3.5. Sortez maintenant de ce conteneur en tapant la commande exit.
## 4- Chercher des images

Pour chercher des images disponibles dans le Docker Hub, on peut utiliser la commande `docker search`. Tentons de chercher le terme “mysql” :
```sh
dev $ docker search mysql
```
Vous devriez observer en sortie une grosse quantité de dépôts dans le Docker Hub. Pour chacun d'eux, on peut observer de nombreuses informations comme le nom, la description, les "stars" et les indicateurs "OFFICIAL" et "AUTOMATED".


Le nom identifie l'image. Comme on peut le remarquer, beaucoup de noms sont sous la forme `<namespace>/<nom_repository>` et représentent les dépôts qui sont fournis par la communauté d'utilisateurs de Docker. D'autres sont seulement sous la forme `<nom_repository>` et correspondent aux repositories contenant des images officielles et supportées par l'équipe Docker.

Les Stars montrent la popularité d'une image. Plus elle en a et plus elle est populaire au sein de la communauté. L'indicateur “OFFICIAL” nous indique si l'image est une image officielle et “AUTOMATED” nous informe si l'image a été construite à partir d’un outil de build automatique du Docker Hub (mais nous y reviendrons plus tard. Ce n’est pas la peine de retenir ces informations pour le moment).


A titre d'exemple, téléchargeons une image officielle.
```sh
dev $ docker image pull mysql
```
Puis:
```sh
dev $ docker container run -i -t --name=mysql mysql /bin/bash
```
Puis à l'intérieur du conteneur :
```
# mysql --version
```
La commande nous renvoie bien la version de MySQL installée. Nous n'avons pas besoin de lancer apt-get ou un autre gestionnaire de paquet, MySQL est déjà présent dans notre image.

Sortez maintenant du conteneur en lançant la commande `exit`.

## 5- Supprimer une image

Pour supprimer une image localement sur notre machine, on utilise la commande `docker image rm`.

Par défaut, Docker ne veut pas supprimer une image qui est liée à un conteneur existant que celui-ci soit arrêté ou non. Il faut donc d’abord supprimer le conteneur mysql.
```sh
dev $ docker container rm mysql
```
On peut ensuite supprimer l’image :
```sh
dev $ docker image rm mysql
```
Observez la sortie : on remarque que chacune des couches du système de fichiers sont supprimées les unes après les autres.

## 6- La pratique pour aller plus loin

> Quelle commande docker permet d'afficher les différentes couches d'une image ? 

> Quelle commande docker permet de supprimer toutes les images non utilisées ? 

> Que permet de faire la commande `docker image save` ?

> Comment puis-je connaître la taille totale d'une image ? 

> Que permet de faire cette commande docker `docker images --no-trunc --digests --filter "dangling=true"` ? 

