# Chapitre 5

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [INTRODUCTION À LA CLI  DOCKER (§5.1)](#introduction-à-la-cli--docker-§51)
        - [Les variables d’environnement Docker (§5.1.2)](#les-variables-denvironnement-docker-§512)
    - [LES COMMANDES SYSTÈME (§5.2)](#les-commandes-système-§52)
        - [docker events / docker system events (§5.2.2)](#docker-events--docker-system-events-§522)
    - [CYCLE DE VIE DES CONTENEURS (§5.3)](#cycle-de-vie-des-conteneurs-§53)
        - [docker create et docker run / docker container create et docker container run (§5.3.9)](#docker-create-et-docker-run--docker-container-create-et-docker-container-run-§539)
    - [INTERACTIONS AVEC UN CONTENEUR DÉMARRÉ (§5.4)](#interactions-avec-un-conteneur-démarré-§54)
        - [docker logs / docker container logs (§5.4.1)](#docker-logs--docker-container-logs-§541)
        - [docker exec / docker container exec (§5.4.2)](#docker-exec--docker-container-exec-§542)
        - [docker attach / docker container attach (§5.4.3)](#docker-attach--docker-container-attach-§543)
        - [docker cp / docker container cp (§5.4.5)](#docker-cp--docker-container-cp-§545)
        - [docker diff / docker container diff  (§5.4.6)](#docker-diff--docker-container-diff--§546)
        - [docker export / docker container export  (§5.4.8)](#docker-export--docker-container-export--§548)
    - [COMMANDES RELATIVES AUX IMAGES (§5.5)](#commandes-relatives-aux-images-§55)
        - [docker save et docker load / docker image save et docker image load (§5.5.6)](#docker-save-et-docker-load--docker-image-save-et-docker-image-load-§556)
    - [INTERACTIONS AVEC LE REGISTRY (§5.6)](#interactions-avec-le-registry-§56)
        - [docker search (§5.6.5)](#docker-search-§565)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### INTRODUCTION À LA CLI  DOCKER (§5.1)

#### Les variables d’environnement Docker (§5.1.2)
```
mkdir -p /vagrant/workdir/chapitre5/alt_docker_config/
tee /vagrant/workdir/chapitre5/alt_docker_config/config.json <<-'EOF'
{
  "psFormat":"table {{.ID}}\\t{{.Image}}\\t{{.Command}}\\t{{.Ports}}\\t{{.Status}}"
}
EOF
export DOCKER_CONFIG=/vagrant/workdir/chapitre5/alt_docker_config/
docker ps
```

Pour réinitialiser l'affichage par défaut, vous pouvez utiliser la commande suivante:
```
unset DOCKER_CONFIG
```

### LES COMMANDES SYSTÈME (§5.2)

> Nous ne reprenons ci-dessous que les lignes de commande complexes et pas les simples ```docker <commande>```.

#### docker events / docker system events (§5.2.2)

Pour cet exemple, vous devez ouvrir un autre terminal sur votre machine virtuelle. Pour ce faire, ouvrez un terminal sur votre hôte, placez-vous dans le répertoire que vous avez créé lors de l'installation de votre environnement de travail (cf [chapitre 3](../chapitre3/README.md#1--installation-de-lenvironnement-docker-linux-avec-vagrant)) et lancez la commande ```vagrant ssh```.


> **ERRATUM** : [docker events / docker system events](../ERRATA.md#docker-events--docker-system-events---§526)

Dans le premier terminal :
```
docker container prune
docker run -d --name nginx nginx
```

Dans le second terminal : 
```
docker events
```

Dans le premier terminal :
```
docker pause nginx
docker unpause nginx
docker stop nginx
docker start nginx
```

> Vous pouvez aussi utiliser l'API REST indiquée en fin de paragraphe avec la commande ```curl https://localhost:2375/events``` à la place de la commande ```docker event``` dans le second terminal. Celle-ci affichera un log similaire à celui-ci (au format JSON):

```
{"status":"kill","id":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","from":"nginx","Type":"container","Action":"kill","Actor":{"ID":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","Attributes":{"image":"nginx","maintainer":"NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e","name":"nginx","signal":"3"}},"scope":"local","time":1648398130,"timeNano":1648398130307689172}
{"status":"die","id":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","from":"nginx","Type":"container","Action":"die","Actor":{"ID":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","Attributes":{"exitCode":"0","image":"nginx","maintainer":"NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e","name":"nginx"}},"scope":"local","time":1648398130,"timeNano":1648398130355676929}
{"Type":"network","Action":"disconnect","Actor":{"ID":"737c13c1b3d9356562154d1a329212b0bdcfe41b58fced5e623ad869907e60e4","Attributes":{"container":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","name":"bridge","type":"bridge"}},"scope":"local","time":1648398130,"timeNano":1648398130402058274}
{"status":"stop","id":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","from":"nginx","Type":"container","Action":"stop","Actor":{"ID":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","Attributes":{"image":"nginx","maintainer":"NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e","name":"nginx"}},"scope":"local","time":1648398130,"timeNano":1648398130413247932}
{"Type":"network","Action":"connect","Actor":{"ID":"737c13c1b3d9356562154d1a329212b0bdcfe41b58fced5e623ad869907e60e4","Attributes":{"container":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","name":"bridge","type":"bridge"}},"scope":"local","time":1648398132,"timeNano":1648398132643159071}
{"status":"start","id":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","from":"nginx","Type":"container","Action":"start","Actor":{"ID":"24bf2d53a1b275b15cbf541c30869cb8190fabb893aee1c0171e59cf7469e4a5","Attributes":{"image":"nginx","maintainer":"NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e","name":"nginx"}},"scope":"local","time":1648398132,"timeNano":1648398132893714377}
```

### CYCLE DE VIE DES CONTENEURS (§5.3)

#### docker create et docker run / docker container create et docker container run (§5.3.9)

```
docker create -name=webserver nginx
```

```
docker start webserver
```

```
docker run -t -i centos:7 /bin/bash
```

### INTERACTIONS AVEC UN CONTENEUR DÉMARRÉ (§5.4)

#### docker logs / docker container logs (§5.4.1)
```
docker run -d --name loop php php -r "while(true){echo \"Log something every 2 sec\n\";sleep(2);}"
docker ps
docker logs --tail 4 loop
```

Pour stopper le conteneur loop:
```
docker stop loop
docker rm loop
```

Ligne de commande utilisant le driver "syslog":
```
docker run -d --log-driver syslog --name loop php php -r "while(true){echo \"Log something every 2 sec\n\";sleep(2);}"
sudo tail -f /var/log/messages
```

#### docker exec / docker container exec (§5.4.2)
```
docker run -d --name webserver nginx
docker exec -t -i webserver /bin/bash
```

#### docker attach / docker container attach (§5.4.3)
```
docker run -d -p 8000:80 --name webserver nginx
docker attach --sig-proxy=false  webserver
```

#### docker cp / docker container cp (§5.4.5)
```
docker run -d -p 8000:80 --name webserver nginx
docker cp webserver:/usr/share/nginx/html/index.html .
echo "Hello World" > index.html
docker cp index.html webserver:/usr/share/nginx/html/index.html
```

#### docker diff / docker container diff  (§5.4.6)
```
docker run -t -i --name exemple centos:7 /bin/bash
docker diff exemple
echo "Hello" > test.txt
docker diff exemple
```

#### docker export / docker container export  (§5.4.8)
```
docker run -d --name webserver nginx
docker export webserver > test.tar
```

Si vous souhaitez regarder ce que contient ce fichier, archive:
```
mkdir extract
tar xvf test.tar --directory extract
ls extract/
```

Vous notez que le répertoire contient l'ensemble du système de fichier du conteneur, soit quelque chose de ce type:
```
[vagrant@localhost ~]$ ls extract/
bin   dev                  docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   media  opt  root  sbin  sys  usr
```

Vous pouvez ensuite utiliser la commande ```docker import``` pour créer une image à partir du fichier "tarball":
```
[vagrant@localhost ~]$ docker import test.tar monimage
sha256:ba29751804efd5f7e4c77f38318079eb93245c554dbaf652a8eedabdd6c3887b
[vagrant@localhost ~]$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
monimage      latest    ba29751804ef   4 seconds ago   140MB
nginxhello    latest    67341a32a437   2 hours ago     93.4MB
php           latest    8b13149eddaa   8 days ago      484MB
nginx         latest    f2f70adc5d89   9 days ago      142MB
hello-world   latest    feb5d9fea6a5   6 months ago    13.3kB
centos        7         eeb6ee3f44bd   6 months ago    204MB
nginx         1.7       d5acedd7e96a   6 years ago     93.4MB
```

### COMMANDES RELATIVES AUX IMAGES (§5.5)

#### docker save et docker load / docker image save et docker image load (§5.5.6)
```
docker save centos:7 > centos.tar
docker load -i=centos.tar
```

### INTERACTIONS AVEC LE REGISTRY (§5.6)

#### docker search (§5.6.5)

```
docker search --filter "is-official=true" php
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://docs.docker.com/engine/reference/commandline/dockerd/

https://docs.docker.com/engine/security/rootless/

https://hub.docker.com/_/php/

https://www.fluentd.org/

https://runc.io/ (redirigé vers https://github.com/opencontainers/runc)

https://hub.docker.com/r/ingensi/play-framework/

https://cloud.google.com/container-registry/

https://azure.microsoft.com/en-us/services/container-registry/

https://quay.io/

