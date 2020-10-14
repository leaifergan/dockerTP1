# Docker - TP3 : Utilisation des conteneurs de manière autonome
> **Objectifs du TP** :
>- Autonomie sur l'utilisation des conteneurs
>

## 1- La mise en pratique autonome

> Que permet de faire la commande suivante `docker container run alpine ls -l` ?
  Lance le container alpine et execute directement un ls -l dans ce containeur
  Liste le contenu du conteneur, résultat au format long (fichiers avec leurs permissions, propriétaires, tailles, et dates de modifications) 

> Quel argument permet de supprimer un container automatiquement une fois celui-ci arrêté ?
  --rm

> Comment partager un volume entre mon hôte local et un conteneur ? 
  docker container run -v ~/abclogs:/var/log/abc alpine
  docker container run -i -t -v /local_dir:/container_dir 

> Que permet de faire la commande docker suivante `docker container exec <nom_conteneur> ls -l` ?
<-ça permet d'executer une commande dans un container en cours d'execution

> Quelle ligne de commande docker permet de créer un conteneur de type serveur web permettant d'héberger votre code situé dans votre répertoire courant ? 
  docker container run -p 80:80 -v /siteweb:/var/www imageavecunserveurweb
  docker container run -d --name webserver -v ./super_projet:/usr/share/nginx/html nginx
  docker container run -v dossierADeployer:dossier destination nom_image
  ./ dossier courant
