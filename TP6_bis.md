# Docker - TP6 BIS : Construction d'images Dockerfile avec Multistaging
> **Objectifs du TP** :
>- Manipuler la création d'images Docker avec son application en Dockerfile et en mode Multistaging
>

## 1 - Création d'une image à l'aide d'un Dockerfile en Multistaging

Nous avons vu dans le cours qu'il était possible et d'ailleurs recommandée de créer des images les plus petites possibles afin d'assurer de la performance et puis surtout éviter les failles potentielles de sécurité. Pour répondre à ce besoin, on utilise le Multistaging. Le concept en soi est simple, on construir une image à partir d'images intermédiaires qu'on va "jeter" afin de ne garder que le nécessaire. 

C'est donc ce que vous allez devoir construire maintenant en autonomie. Afin de réaliser ce TP vous allez devoir réaliser les actions suivantes : 

1 - Créer une applicatin en Node.js qui permet de faire un "HelloWorld from Node" lorsqu'on accède à la racine de votre application. Pour les besoins du TP vous pouvez aussi prendre n'importe quel template de base qui permet d'afficher une page d'accueil.

2 - Créer un fichier `Dockerfile` dans lequel vous aurez un premier stage basé sur une image officielle "node" de DockerHub

3 - Ajouter le fichier `package.json` sur l'image

4 - Installer tous les modules nodes en dependency et/ou devDependency (choix libre en fonction de votre configuration)

5 - Ajouter votre code applicatif sur l'image 

6 - Compiler votre code (à minima), lancer vos tests, linters (ou autres)

7 - Créer un nouveau stage avec une image de base adéquate

8 - Installer les modules nécessaires au runtime de l'application (on utilisera généralement `npm install --only=production`

9 - Exposer le port d'écoute de votre application 

10 - Définir le point d'entrée (la commande à lancer au démarrage) de votre image

11 - Builder votre Dockerfile de façon à en faire une image que vous allez devoir pousser sur votre compte DockerHub 

12 - Pousser l'image sur DockerHub 

13 - Créez ensuite un conteneur avec votre image poussée sur DockerHub : `docker container run -d -p [port_src]:[port_dest] $UTILISATEUR/[image_name]`

14 - Accéder sur votre navigateur à l'url : `localhost:[port_src]` afin de voir s'afficher la page d'accueil que vous avez créée au début du TP. 

BONUS : Réalisez à nouveau le même TP sans utiliser le Multistaging afin de comparer les tailles des deux images que vous aurez créé (la version Multistaging et celle sans, vous devriez y voir une différence)

BONUS : Voyez-vous d'autres optimisations possibles sur votre Dockerfile ? Nous échangerons ensemble afin de challenger un peu votre code. 

Pour rappel, voici les instructions de Dockerfile :

`FROM` est une directive permettant de qualifier l'image de base utilisée. Pas de mystère, on utilise une version d’AlpineLinux.

`LABEL` est une directive qui permet d’ajouter des metadata à l’image. Ici, cela nous permet de définir la personne maintenant l'image. **Remplacez par vos nom et prénoms**.

`RUN` est l'une des directives principales d'un Dockerfile et permet d'exécuter des instructions qui vont permettre de construire notre image. Ici, on installe les dépendances de notre application avec pip.

`COPY` est une directive qui permet d’ajouter des fichiers provenant de notre host dans une image. Une option au comportement similaire est ADD, mais elle n’est pas conseillée vu son caractère “automagique”. Elle décompresse des fichiers et marche avec n’importe quel type de ressource, ce qui rend difficile de prévoir son comportement...

`WORKDIR` est une directive qui permet de changer le working directory par défaut de notre image.

`USER` est une directive permettant de spécifier quel sera l’utilisateur courant. Il est important de spécifier un utilisateur non-root (avec un uid supérieur ou égal à 1000) pour des raisons de sécurité.

`ENTRYPOINT` et `CMD` permettent de définir la commande qui est lancée au démarrage de notre conteneur. On parlera des différences entre ces deux directives plus tard dans la formation.

Vous pouvez builder votre image en utilisant la commande :

`docker image build .` # Attention, il faudra ajouter l'argument "-t" si vous souhaitez lui donner un nom afin de la pousser sur votre Registry Docker.

