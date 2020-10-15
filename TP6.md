# Docker - TP6 : Construction d'images en Dockerfile
> **Objectifs du TP** :
>- Utiliser un registre afin de stocker des images Docker
>- Comprendre comment est construite une image Docker avec son application en Dockerfile
>

## 1- Création d'une image à l'aide d'un Dockerfile

Nous savons que deux méthodes sont possibles pour construire une image. La deuxième, que nous allons voir maintenant, est la méthode recommandée. Nous allons décrire comment réaliser un petit fichier permettant de construire une image à l'aide d'une suite d'instructions. Ce fichier s'appelle le Dockerfile.

Nous allons créer notre propre application, et cette fois-ci à l'aide d'un Dockerfile !

Veillez à quitter le conteneur dans lequel vous êtes. Puis préparez à travailler dans un répertoire vide car on y aura plusieurs à mettre :

Dans le fichier `app.py`, copiez le code suivant (attention à l’indentation) :
```py
# Docker-app/app.py
from flask import Flask
from http import HTTPStatus
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Docker!"

@app.errorhandler(HTTPException)
def handle_http_exception(exception: HTTPException):
    return exception.description, exception.code

@app.errorhandler(Exception)
def handle_exception(exception: Exception):
    return HTTPStatus.INTERNAL_SERVER_ERROR.description, HTTPStatus.INTERNAL_SERVER_ERROR
```

Ce code représente une application que l’on fera évoluer pour montrer toutes les capacités de Docker. Il utilise un framework Python qui s’appelle `Flask`. Il nous permettra d'ajouter des fonctionnalités de façon assez simple, sans avoir besoin de rajouter beaucoup de code.

Pour l’instant l’application est très basique : elle répond sur la route racine `/` par `Hello Docker` et un code `HTTP 200`.

Pour s'exécuter, l'application nécessite des modules comme `Flask`. On utilisera aussi `gunicorn`, un serveur Python WSGI HTTP. Nous allons mettre toutes ces dépendances dans un fichier de prérequis standard en Python : `requirements.txt`. Dans ce fichier, mettez le contenu suivant :
```sh
# Docker-app/requirements.txt
Flask==1.0.2
gunicorn==19.9.0
```

Éditez également le fichier `config.py` avec le contenu suivant :
```py
# Docker-app/config.py
import os

bind = "0.0.0.0:8000"
worker_class = "gthread"
threads = int(os.getenv('GUNICORN_CORES','1')) * 2 + 1
```
Utilisez l'éditeur vim (ou le Web IDE) pour créer un fichier Dockerfile. Dans ce fichier, ajoutez le contenu suivant :

```Dockerfile
# Docker-app/Dockerfile
FROM python:3.7-alpine
LABEL maintainer="<Nom> <Prénom>"

# Create and change working directory
WORKDIR /app

# Add application requirements
COPY requirements.txt .

# Install requirements
RUN pip install -r requirements.txt

# Add application
COPY app.py .
COPY config.py .

# Create a specific user to run the Python application
RUN adduser -D my-user -u 1000
USER 1000

# Launch application
ENTRYPOINT ["gunicorn"]
CMD ["--config", "config.py", "app:app"]
```

Prenons le temps d'étudier ces commandes :

`FROM` est une directive permettant de qualifier l'image de base utilisée. Pas de mystère, on utilise une version d’AlpineLinux.

`LABEL` est une directive qui permet d’ajouter des metadata à l’image. Ici, cela nous permet de définir la personne maintenant l'image. **Remplacez par vos nom et prénoms**.

`RUN` est l'une des directives principales d'un Dockerfile et permet d'exécuter des instructions qui vont permettre de construire notre image. Ici, on installe les dépendances de notre application avec pip.

`COPY` est une directive qui permet d’ajouter des fichiers provenant de notre host dans une image. Une option au comportement similaire est ADD, mais elle n’est pas conseillée vu son caractère “automagique”. Elle décompresse des fichiers et marche avec n’importe quel type de ressource, ce qui rend difficile de prévoir son comportement...

`WORKDIR` est une directive qui permet de changer le working directory par défaut de notre image.

`USER` est une directive permettant de spécifier quel sera l’utilisateur courant. Il est important de spécifier un utilisateur non-root (avec un uid supérieur ou égal à 1000) pour des raisons de sécurité.

`ENTRYPOINT` et `CMD` permettent de définir la commande qui est lancée au démarrage de notre conteneur. On parlera des différences entre ces deux directives plus tard dans la formation.

On peut ensuite créer notre image en utilisant la commande :

`docker image build .`

Cett commande devrait alors vous générer une nouvelle image, trouvable via la commande 

`docker images` 