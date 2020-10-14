# Docker - TP4 : La création d'images
> **Objectifs du TP** :
>- Création d'images et utilisation de la Registry Docker Hub
>

## 1- Construction de notre propre image

Maintenant que nous avons vu comment récupérer des images toutes prêtes avec des programmes préinstallés, créons nous même notre propre image personnalisée !
Il existe deux façons de créer notre propre image :

- Grâce à la commande `docker container commit` qui va créer une nouvelle couche de système de fichiers avec les nouveaux fichiers d'un conteneur (par exemple on installe MySQL au dessus d'une image de base ubuntu).

- Avec la commande `docker image build`. Cette commande est différente car une image est construite via une description dans un fichier appelé Dockerfile qui contient une suite d'instruction à exécuter pour construire une image. Par exemple, on peut décrire dans le fichier que l'on doit installer MySQL à partir d'une image de base Ubuntu mais ce n'est pas nous qui interagissons directement avec un conteneur et sauvegardons les changements. C'est Docker lui même qui se charge d'installer MySQL pour nous et qui sauvegarde une nouvelle image !

## 2- Utilisation d’un registre pour faire un commit des changements d'un conteneur

Nous allons maintenant faire un **commit** des changements d'un conteneur que l'on va lancer grâce à l'instruction `docker container commit`. Lançons un conteneur Ubuntu classique.
```sh
dev $ docker container run --name=vimubuntu -i -t ubuntu /bin/bash
```
Vous devriez obtenir, comme d’habitude, une invite de commande de la forme suivante :
```
root@adf131ef9a23:/#
```
Votre conteneur peut être identifié soit par le nom que vous lui avez donné au travers du flag `--name` ou grâce au nombre hexadecimal après root@, ici adf131ef9a23 : c’est l’identifiant court du conteneur.
Effectuons maintenant quelques changements dans notre conteneur.
```
# apt-get update
# apt-get -y install vim
```
Nous venons d'installer vim sur notre conteneur Ubuntu. Cependant, nous savons que si nous tuons le conteneur et que nous le supprimons, nous n'aurons aucun moyen de repartir de ce point sans avoir à retaper ces commandes et télécharger de nouveau le paquet Vim. Pour éviter cela, nous allons faire un commit du conteneur et pousser son image dans un dépôt de notre registre privé ! Tapez exit et le moment est venu de vous créer un compte sur https://hub.docker.com retenez ensuite le nom d'utilisateur car vous en aurez besoin...

Sur votre terminal, trouvez la commande qui vous permet de vous connecter à la registry DockerHub et lorsque la connexion sera bien effectuée vous pourrez passer à la prochaine étape qui sera de créer une variable d'environnement sur votre machine hôte `$UTILISATEUR`à entrer manuellement et qui a pour valeur le nom d'utilisateur que vous avez créé sur le hub.docker.com

Lancez ensuite la commande suivante en :
```sh
dev $ docker container commit vimubuntu $UTILISATEUR/vimubuntu
```

En retour de la commande, Docker nous donne un identifiant :
```sh
sha256:f22ba10fe40b9d87fb51dfaf7a0e4d5e863e6a3eb6edb1b2feb0516f82ad3443
```
C'est l'identifiant de l'image que nous avons construite à partir des changements effectués dans le conteneur.

Pour vérifier que nous avons bien créé une nouvelle image, supprimons notre ancien conteneur.
```sh
dev $ docker container rm vimubuntu
```
Listons les images pour essayer de retrouver la nôtre:
```sh
$ docker image ls
REPOSITORY                     TAG       IMAGE ID            CREATED             SIZE
$UTILISATEUR/vimubuntu   latest    fe12ceb6e74c        8 minutes ago       186 MB
```
Notre image est bien listée comme $UTILISATEUR/vimubuntu (où la variable est remplacée par la valeur correspondante à votre environnement). Nous pouvons maintenant l’envoyer au registre avec la commande suivante :
```sh
dev $ docker image push $UTILISATEUR/vimubuntu
```
 Relançons un autre conteneur grâce à cette image :
```sh
dev $ docker container run -i -t $UTILISATEUR/vimubuntu /bin/bash
```
Tapez la commande vim dans le shell. On remarque que vim est déjà installé sur le conteneur. Nous avons sauvegardé nos changements avec succès!

Vous pouvez quitter vim en tapant sur la touche ESC puis en tapant :q! puis sortir du conteneur en lançant la commande exit.
