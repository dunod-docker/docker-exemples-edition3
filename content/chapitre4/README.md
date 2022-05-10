# Chapitre 4

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [Créer et démarrer un conteneur (commande run) (§4.1.1)](#créer-et-démarrer-un-conteneur-commande-run-§411)
    - [Stopper un conteneur (commande stop ou kill) ($4.1.2)](#stopper-un-conteneur-commande-stop-ou-kill-412)
        - [Identifier un conteneur](#identifier-un-conteneur)
        - [Résultat de la commande stop](#résultat-de-la-commande-stop)
        - [Redémarrer un conteneur stoppé](#redémarrer-un-conteneur-stoppé)
    - [Détruire un conteneur (commande rm) (§4.1.3)](#détruire-un-conteneur-commande-rm-§413)
    - [La commande create (§4.1.4)](#la-commande-create-§414)
    - [Connexion en mode terminal (§4.2.1)](#connexion-en-mode-terminal-§421)
    - [Créer un volume (§4.2.2)](#créer-un-volume-§422)
    - [Monter un répertoire de notre hôte dans un conteneur (§4.2.3)](#monter-un-répertoire-de-notre-hôte-dans-un-conteneur-§423)
    - [La configuration des ports IP (§4.2.4)](#la-configuration-des-ports-ip-§424)
    - [Lister les images (§4.3.1)](#lister-les-images-§431)
    - [Charger et effacer les images (§4.3.2)](#charger-et-effacer-les-images-§432)
    - [Créer une image à partir d’un conteneur (§4.3.3)](#créer-une-image-à-partir-dun-conteneur-§433)
    - [Programmer la création des images (§4.4.2)](#programmer-la-création-des-images-§442)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### Créer et démarrer un conteneur (commande run) (§4.1.1)

```
docker run -p 8000:80 -d nginx
```

### Stopper un conteneur (commande stop ou kill) ($4.1.2)

#### Identifier un conteneur

```
docker ps
```

#### Résultat de la commande stop

```
docker stop <id du conteneur>
```

Remplacez "\<id du conteneur\>" par l'identifiant approprié comme indiqué dans le livre

```
sudo ls /var/lib/docker/containers/
```

#### Redémarrer un conteneur stoppé

```
docker ps -a
```

```
docker start <id du conteneur>
```

Remplacez "\<id du conteneur\>" par l'identifiant approprié comme indiqué dans le livre

### Détruire un conteneur (commande rm) (§4.1.3)

```
docker rm <id du conteneur>
```

```
docker stop <id du conteneur>
docker rm <id du conteneur>
```

Remplacez "\<id du conteneur\>" par l'identifiant approprié comme indiqué dans le livre

### La commande create (§4.1.4)

```
docker create -p 8000:80 nginx
docker ps -a
```

```
docker start <id du conteneur>
```

### Connexion en mode terminal (§4.2.1)

```
docker run -d -p 8000:80 --name webserver nginx
```

```
docker exec -t -i webserver /bin/bash
```

Remplacez "\<id du conteneur\>" par l'identifiant approprié comme indiqué dans le livre

```
hostname
echo "Je suis dedans" > /usr/share/nginx/html/index.html
exit
```

### Créer un volume (§4.2.2)

```
docker run -p 8000:80 --name webserver -d -v /usr/share/nginx/html nginx
docker inspect webserver
```

```
ls /var/lib/docker/volumes/d453c27b35d9e1e89442e35fffe6fd775f3d2b0ff5086ed4eac0641184ebff80/_data
```

Remplacez "d453c27b35d9e1e89442e35fffe6fd775f3d2b0ff5086ed4eac0641184ebff80" par la valeur que vous avez trouvé. Cf. les explications dans le livre.

```
echo "Je modifie un volume" > /var/lib/docker/volumes/d453c27b35d9e1e89442e35fffe6fd775f3d2b0ff5086ed4eac0641184ebff80/_data/index.html
```

### Monter un répertoire de notre hôte dans un conteneur (§4.2.3)

```
mkdir -p /vagrant/workdir/chapitre4
docker run -p 8000:80 --name webserver -d -v /vagrant/workdir/chapitre4:/usr/share/nginx/html nginx
```

```
cd /vagrant/workdir/chapitre4
echo "Un fichier sur le host" > index.html
```

### La configuration des ports IP (§4.2.4)

```
docker run -d -p 8000:80 --name webserver nginx
docker inspect webserver
```

Il se peut que l'attribut ```HostIp``` soit vide à la place de contenir "0.0.0.0" comme indiqué dans le livre.

```
    "PortBindings": {
        "80/tcp": [
            {
                "HostIp": "",
                "HostPort": "8000"
            }
        ]
    },
```

```
docker rm -f webserver
docker run -P --name webserver -d nginx
docker ps
```

```
curl http://localhost:<port IP>
```

Remplacez "\<port IP\>" par la valeur trouvée avec ```docker ps```


### Lister les images (§4.3.1)

```
docker images
```

```
docker images -a
```

### Charger et effacer les images (§4.3.2)

```
docker rmi nginx
```

```
docker rm -f $(docker ps -a -q)
```

Notez que la commande précédente est équivalente à ```docker container prune``` :
```
[vagrant@localhost ~]$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

```
docker rmi nginx
```

```
docker pull nginx:1.7
docker run -p 8000:80 --name webserver17 -d nginx:1.7
docker run -p 8001:80 --name webserver121 -d nginx
```

> ATTENTION, utilisez bien les ports 8000 et 8001. En effet, souvenez-vous que tous les ports ne sont pas ouverts entre la machine Vagrant et votre hôte (cf. [Procédure d'installation](../chapitre3/README.md#1--installation-de-lenvironnement-docker-linux-avec-vagrant)).

Pour effacer les conteneurs à l'issue de l'exercice, utilisez:
```
docker stop webserver17 webserver121
docker container prune
```

### Créer une image à partir d’un conteneur (§4.3.3)

```
docker run -p 8000:80 --name webserver -d nginx
docker exec -i -t webserver /bin/bash
```

Les commandes suivantes sont exécutées à l'intérieur du conteneur:
```
echo "Hello world" > /usr/share/nginx/html/index.html
exit
```

```
docker commit webserver nginxhello
docker images
```

```
docker run -p 8001:80 --name webserverhello -d nginxhello
```

### Programmer la création des images (§4.4.2)

N'oubliez par de créer un répertoire vide dans votre espace de travail (sous /vagrant par exemple).

```
echo "Un fichier qui dit \"Hello\"" > index.html
```

Créez un fichier Dockerfile contenant le code suivant:
```
FROM nginx:1.7
COPY index.html /usr/share/nginx/html/index.html
```

```
docker build -t nginxhello .
docker images
```

```
docker run -p 8000:80 --name webserver -d nginxhello
```

```
docker history nginxhello
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

http://nginx.org/

https://www.ietf.org/rfc/rfc4122.txt

https://github.com/moby/moby/blob/master/pkg/namesgenerator/names-generator.go

https://www.json.org/

