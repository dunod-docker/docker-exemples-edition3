# Chapitre 11

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [IDENTIFICATION DES PROBLÉMATIQUES (§11.1)](#identification-des-problématiques-§111)
        - [Création et configuration de l’environnement de travail (§11.1.2)](#création-et-configuration-de-lenvironnement-de-travail-§1112)
    - [INSTANCIATION DES CONTENEURS (§11.2)](#instanciation-des-conteneurs-§112)
        - [Préparation des agents serveur et client (§11.2.1)](#préparation-des-agents-serveur-et-client-§1121)
        - [Déploiement avec Nomad (§11.2.2)](#déploiement-avec-nomad-§1122)
    - [DÉCOUVERTE DE SERVICE ET CONNECTIVITÉ (§11.3)](#découverte-de-service-et-connectivité-§113)
        - [Installation du cluster Swarm (§11.3.2)](#installation-du-cluster-swarm-§1132)
    - [GESTION DE CONFIGURATION (§11.6)](#gestion-de-configuration-§116)
        - [Config Docker (§11.6.1)](#config-docker-§1161)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

> **IMPORTANT** : Dans ce chapitre nous utilisons un environnement à plusieurs VM pour simuler plusieurs machines. 
> Vous devriez éteindre la machine virtuelle qui vous a servie dans les chapitres précédents en quittant le terminal et en lançant la commande ```vagrant halt``` dans le répertoire d'installation créé au [chapitre 3](../chapitre3/README.md).

> Le ```Vagrantfile``` pour cet environnement se trouve dans le répertoire ```docker-exemples-edition3-code/chapitre11```.

### IDENTIFICATION DES PROBLÉMATIQUES (§11.1)

#### Création et configuration de l’environnement de travail (§11.1.2)

Placez-vous dans le répertoire ```docker-exemples-edition3-code/chapitre11```.
```
vagrant up
```

> Vous trouverez un exemple de log complet de l'instruction ```vagrant up``` dans le fichier [vagrant-up.log](vagrant-up.log).

### INSTANCIATION DES CONTENEURS (§11.2)

#### Préparation des agents serveur et client (§11.2.1)

Démarrage du noeud Nomad serveur:
```
sudo nomad agent -server -bootstrap-expect=1  -bind=$(ip -f inet addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1) -data-dir=/opt/nomad/data -log-level INFO
```

Notez les adresses IP de chaque machine:
```
ip -f inet addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1
```

Démarrez un client nomade sur chaque noeud client:
```
sudo nomad agent -client -data-dir=/opt/nomad/data -servers=<<addresse IP du noeud serveur>> -network-interface=eth1
```

Lancez sur le noeud serveur:
```
sudo nomad node status -address=http://<<addresse IP du noeud serveur>>:4646
```

#### Déploiement avec Nomad (§11.2.2)

> **Note** : Le fichier "hello-front.nomad" se trouve, à l'intérieur des machines virtuelle, dans le répertoire ```/vagrant```.

```
nomad job run -address=http://<<addresse IP du noeud serveur>>:4646 hello-front.nomad
```
> Vous trouverez un exemple de log de l'exécution de cette commande dans le fichier [job-hello-front.log](job-hello-front.log).

> **Note** : pour tester la destruction de l'instance lancée sur le node-client-1, vous devez ouvrir un autre terminal sur ce dernier.

Listez les conteneurs sur ```node-client-1```:
```
sudo docker ps
```

Vous pouvez ainsi visualiser le conteneur tournant sur cette machine:
```
[vagrant@node-client-1 ~]$ sudo docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                                        NAMES

495c49d4bf7e   flask-app:local   "python3 -m flask ru…"   9 minutes ago   Up 9 minutes   172.28.128.18:8080->5000/tcp, 172.28.128.18:8080->5000/udp   flask-app-2467221d-e89e-de54-9438-646197be0b90
```

Détruisez le conteneur:
```
sudo docker kill flask-app-2467221d-e89e-de54-9438-646197be0b90
```

### DÉCOUVERTE DE SERVICE ET CONNECTIVITÉ (§11.3)

Cleanup Nomad:
```
sudo nomad job stop -purge  -address=http://<<addresse IP du noeud serveur>>:4646 hello-app
```

#### Installation du cluster Swarm (§11.3.2)

Démarrage du master:
```
sudo docker swarm init --advertise-addr $(ip -f inet addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1)
```

```
sudo docker node ls
```

Démarrage du service hello-front:
```
sudo docker service create --replicas=2 --name hello-front --publish published=8080,target=5000 flask-app
```
> Vous trouverez un exemple de log de l'exécution de cette commande dans le fichier [swarm-hello-front-start.log](swarm-hello-front-start.log).

```
sudo docker service ps hello-front --format "table {{.ID}} {{.Name}} {{.Node}} {{.CurrentState}}"
```

Destruction du service:
```
sudo docker service rm hello-front
```

Réseaux disponibles:
```
docker network ls
```

Virtualisation de l’accès aux services:
```
sudo docker network create -d overlay hello-overlay
sudo docker service create --replicas=2 --name hello-front --network=hello-overlay --publish published=8080,target=5000 flask-app
sudo docker service create --replicas=1 --name hello-back --network=hello-overlay flask-app
```
> Vous trouverez un exemple de log de l'exécution de ces commandes dans le fichier [swarm-hello-front-start.log](swarm-hello-front-start.log).

```
sudo docker service ps --format "table {{.ID}} {{.Name}} {{.Node}} {{.CurrentState}}" hello-front
```

```
sudo docker service ps --format "table {{.ID}} {{.Name}} {{.Node}} {{.CurrentState}}" hello-back
```

```
sudo docker service scale hello-back=2
```

### GESTION DE CONFIGURATION (§11.6)

#### Config Docker (§11.6.1)

```
echo "John" | sudo docker config create mon_nom -
```

```
sudo docker service update --config-add mon_nom hello-front
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu

https://palletsprojects.com/p/flask/

https://www.consul.io/

https://www.haproxy.com/products/aloha-virtual-load-balancer/

https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/

https://istio.io/

https://linkerd.io/

https://www.envoyproxy.io/

https://www.consul.io/docs/connect/gateways/ingress-gateway

https://www.vaultproject.io/

https://github.com/hashicorp/envconsul

