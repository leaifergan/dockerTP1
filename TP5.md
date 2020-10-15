# Docker - TP5 : Utilisation des networks de manière autonome
> **Objectifs du TP** :
>- Autonomie sur l'utilisation des networks
>

## 1- La mise en pratique autonome

> Que permet de faire la commande suivante `docker network inspect ngnet` ?
  Afficher des informations détaillées sur le réseau ngnet

> Quel argument permet de connecter un conteneur que l'on va lancer à un network créé préalablement ? 
  --connect
  --network (à la création du réseau)
  
> Quels sont les 3 principaux types de networks qui existent ? 
  Bridge, Host, None

> Pour quel besoin pourrait-on utiliser la notion de networks avec Docker ?
  Pour que les conteneurs Docker puissent communiquer entre eux mais aussi avec le monde extérieur via la machine hôte. Avec le  réseau none permet de bloquer les entrées et sorties.

> Quelle suite de commandes Docker permet de créer un network de type `bridge` personnalisé, de connecter ce network au conteneur et de vérifier que ce network est bien utilisé par une nouvelle carte réseau eth1 ? (eth0 étant le réseau par défaut bridge déjà géré par Docker)
Vous devrez pour ça obligatoirement utiliser un nombre exact de 3 commandes Docker. 

docker network create --driver bridge mon-bridge
docker run --network=<network-name>
docker network connect nom_network nom_container nom_host OU docker container run --network myhost mycontainer
ip addr show nom_network
docker network inspect my_host 
docker container inspect nom-container
