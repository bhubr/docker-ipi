# Atelier Docker et CI/CD

## Rappels sur Docker

Docker est une plate-forme permettant de lancer des applications dans des _conteneurs_.

Selon 451 Research, cité par Wikipedia :

> Docker est un outil qui peut empaqueter une application et ses dépendances dans un conteneur isolé, qui pourra être exécuté sur n'importe quel serveur

Docker est un outil de _conteneurisation_, à ne pas confondre avec la _virtualisation_.

Les outils de virtualisation (VMWare, VirtualBox, etc.) :

* permettent de lancer des _machines virtuelles_, 
* lesquelles sont préalablement configurées (nombre de CPU, quantité de RAM, interfaces réseau...),
* et dans lesquelles on doit installer un OS (appelé "OS invité" par opposition à l'"OS hôte", celui de la machine qui exécute le logiciel de virtualisation).

Du fait qu'on lance un OS complet pour pouvoir lancer une application au-dessus, la virtualisation est gourmande en ressources.

Par opposition, la conteneurisation :

* ne repose pas sur des machines virtuelles "isolées" de la machine hôte,
* ni sur le lancement d'un OS complet,
* permet de lancer des applications dans des conteneurs,
* utilisant noyau de l'OS hôte,
* relativement isolés du reste du système hôte,
* n'ayant pas par défaut de limites matérielles (usage de RAM par ex.) autres que celles disponibles sur le système hôte.

La conteneurisation est, par comparaison à la virtualisation, économe en ressources.

![virtualization vs contenerization](https://www.netapp.com/media/Screen-Shot-2018-03-20-at-9.24.09-AM_tcm19-56643.png?v=85344)

> À noter : Docker n'est pas le seul système de conteneurisation. Il existe une certaine interopérabilité entre différents systèmes, via l'utilisation d'_images_ qui respectent un format standardisé.

### Cas d'utilisation et avantages

* En "packageant" une application et ses dépendances dans une "image" exécutable n'importe où, on s'affranchit de problèmes comme, par exemple, l'incompatibilité d'une application avec certaines bibliothèques (ou versions de ces bibliothèques) installées sur le système cible.
* De fait, on peut, dans une équipe de développeurs, créer, via Docker, un environnement de travail standardisé, que tout le monde va pouvoir récupérer et utiliser tel quel, quel que soit l'OS utilisé.
* Plus intéressant encore, Docker rend le déploiement d'applications beaucoup plus aisé. À partir du moment où Docker est installé sur un serveur, on pourra lancer des applications serveur web, des bases de données, etc. dans Docker, sans casse-têtes de configuration.

## Images et conteneurs

Parmi les concepts importants :

* **Image** : "paquet" contenant l'application et ses dépendances (bibliothèques / outils dont elle a besoin). Si on prend la métaphore du développement logiciel suivant le paradigme de la P.O.O., on pourrait comparer une image à une classe.
* **Conteneur** : processus démarré à partir de l'image, autrement dit, l'application en fonctionnement. Suivant la même comparaison, on pourrait comparer un conteneur à une instance d'une classe.

On peut lancer plusieurs conteneurs à partir d'une même image. Petit exemple pour illustrer, qui permettra au passage de (re)voir certaines commandes de base de Docker.

### Récupérer une image

Les applications "empaquetées" avec Docker sont le plus souvent stockées sur un _registre_ de conteneurs. Le plus connu est le [Docker Hub](https://hub.docker.com/), mais il en existe bien d'autres :

* GHCR (GitHub Container Registry),
* l'équivalent chez les concurrents de GitHub (GitLab, Atlassian),
* les registres fournis par les acteurs du cloud (AWS, Azure, GCP),
* voire des registres auto-hébergés, privés, accessibles au sein d'une organisation uniquement

Nous allons récupérer une image d'une application très simple, le "Hello World" de Docker, via la commande `docker pull` :

```
docker pull hello-world:latest
```

Ce qui produit l'affichage suivant :

```
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:13e367d31ae85359f42d637adf6da428f76d75dc9afeb3c21faea0d976f5c651
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

Docker a constaté que l'image `hello-world`, avec le "tag" (~version) `latest`, n'était pas disponible localement, et l'a téléchargée.

### Lancer un conteneur

Une fois l'image téléchargée, on peut lancer un conteneur à partir de cette image, via `docker run` :

```
docker run hello-world:latest
```

Ce qui produit cet affichage :

```

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

> Notez que si on n'avait pas préalablement exécuté `docker pull`, Docker aurait téléchargé l'image de toute façon. Ce qui n'enlève pas l'utilité de cette commande, qui permet de récupérer la dernière version de l'image disponible, pour peu qu'une nouvelle version ait été publiée depuis le dernier `pull`.

### Lister les conteneurs

Essayez la commande `docker ps -a` (`docker ps` pour afficher les conteneurs Docker, `-a` pour inclure ceux qui ne sont pas/plus en fonctionnement). Résultat :

```
CONTAINER ID   IMAGE                 COMMAND       CREATED          STATUS                       PORTS      NAMES
37aefecd68e8   hello-world:latest    "/hello"      3 minutes ago    Exited (0) 4 minutes ago                boring_solomon
```

Si vous relancer la commande `docker run hello-world:latest` suivie de `docker ps -a`, vous verrez une deuxième ligne apparaître, montrant qu'on peut bien lancer plusieurs conteneurs à partir d'une même image.

### Supprimer conteneurs et images

Il est utile de faire le ménage dans les conteneurs qui ne servent plus (ici l'application Hello World a quitté immédiatement, ce qui n'est pas le cas d'autres applications comme des serveurs web).

On peut supprimer un conteneur avec `docker rm` suivi soit de l'ID (ou de quelques caractères de l'ID) ou du nom du conteneur. Ainsi, ces 3 commandes auront le même effet :

* `docker rm 37aefecd68e8`
* `docker rm 37a`
* `docker rm boring_solomon`

Quant une image ne nous sert plus, on peut la supprimer. Listez les images avec `docker image ls`. Résultat :

```
REPOSITORY          TAG            IMAGE ID       CREATED             SIZE
hello-world         latest         feb5d9fea6a5   9 months ago        13.3kB
```

Vous pouvez supprimer une image via la combinaison repository + tag ou via son ID :

* `docker rmi hello-world:latest`,
* ou `docker rmi feb5d9fea6a5`

## Conteneuriser une application

Récupérer des images "prêtes à l'emploi" du Docker Hub ou d'un autre registre est une chose. Mais comment "packager" ses propres applications dans des images Docker ?

