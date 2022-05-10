# Chapitre 8

> **IMPORTANT** : veuillez vous reporter au [chapitre 3](../chapitre3/README.md) pour installer le dépôt de code dans votre machine virtuelle vagrant. Les répertoires comme ```/application-exemple/``` se trouve dans le répertoire ```/vagrant/docker-exemples-edition3-code```. Le code pour ce chapitre se trouve donc dans ```/vagrant/docker-exemples-edition3-code/chapitre8```.

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [RÉSEAU ET CONTENEURS (§8.2)](#réseau-et-conteneurs-§82)
        - [Le réseau bridge par défaut (§8.2.4)](#le-réseau-bridge-par-défaut-§824)
        - [Créer un réseau personnalisé de type (§8.2.5)](#créer-un-réseau-personnalisé-de-type-§825)
        - [Lien entre les conteneurs,l’hôte et les réseaux auxquels il est connecté (§8.2.6)](#lien-entre-les-conteneurslhôte-et-les-réseaux-auxquels-il-est-connecté-§826)
    - [PERSISTANCE : BIND MOUNTS ET VOLUMES (§8.3)](#persistance--bind-mounts-et-volumes-§83)
        - [Les deux types de systèmes de fichiers persistants (§8.3.2)](#les-deux-types-de-systèmes-de-fichiers-persistants-§832)
    - [CONFIGURATION D’APPLICATION (§8.5)](#configuration-dapplication-§85)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### RÉSEAU ET CONTENEURS (§8.2)

> **IMPORTANT** : Les scripts utilisés dans ce chapitre se trouvent dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre8/network```.

#### Le réseau bridge par défaut (§8.2.4)
```
docker network ls
```

Pour exécuter le script suivant vous devez vous placer dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre8/network```
```
./bridge_network/run_bridge_network.sh
```
```
docker network inspect bridge
```

Interface veth:
```
ifconfig docker0
```
```
sudo ip address show master docker0
```

Pensez à noter le numéro de processus:
```
docker inspect -f '{{.State.Pid}}' centos-test-1
```

Pensez à remplacer 2575 par le numéro de processus noté précédemment:
```
sudo nsenter -t 2575 -n ip addr show type veth
```

Communication entre conteneurs via le bridge par défaut:
```
docker exec -ti centos-test-1 /bin/bash
```
L'instruction ci-dessus ouvre un terminal à l'intérieur du conteneur.

```
yum install -y nmap
nc -l 5000
```

> **ATTENTION** : pour ouvrir un autre terminal vous devez revenir sur votre hôte (celui sur lequel est exécuté la machine virtuelle) et lancer la commande ```vagrant ssh```.

```
./bridge_network/clean_bridge_network.sh
```

#### Créer un réseau personnalisé de type (§8.2.5)
```
./dedicated_network/run_dedicated_network.sh
```
```
docker network ls
```
```
docker network inspect test --format '{{.Containers}}'
```
```
docker network inspect bridge --format '{{.Containers}}'
```
```
docker exec -ti busybox-1 /bin/sh
```
```
docker network connect bridge busybox-1
```
```
docker exec -ti busybox-1 /bin/sh
```
```
./dedicated_network/clean_dedicated_network.sh
```

#### Lien entre les conteneurs,l’hôte et les réseaux auxquels il est connecté (§8.2.6)

Transfert des paquets IP:
```
sysctl net.ipv4.conf.all.forwarding
```

Accès aux réseaux extérieurs:
```
sudo iptables -t nat -L -n
```

Accès à l’hôte depuis un conteneur : l’option « host »:
```
./host_network/run_host_network.sh
```
```
docker exec -ti centos-test /bin/bash
```
```
yum install -y nmap
nc -l 5000
```

> **ATTENTION** : pour ouvrir un autre terminal vous devez revenir sur votre hôte (celui sur lequel est exécuté la machine virtuelle) et lancer la commande ```vagrant ssh```.

```
netstat -vatn | grep 5000
```
```
./host_network/clean_host_network.sh
```

Accès à l’hôte depuis un conteneur : l’option « gateway »:
```
./access_host/run_access_host.sh
```

> **ERRATUM** : Dans le livre cette commande fait référence à une image nommée ```centos-host``` alors que le bon nom est ```centos-test```.
```
docker exec -ti centos-test /bin/bash
```
```
yum install -y net-tools nmap openssh-clients
```
```
echo $(route -n | awk '/UG[ \t]/{print $2}')
```

```
./access_host/clean_access_host.sh
```

### PERSISTANCE : BIND MOUNTS ET VOLUMES (§8.3)

#### Les deux types de systèmes de fichiers persistants (§8.3.2)

Bind mount, rappel:
```
docker system prune -f
```
```
docker run --rm -ti --name=centos-test -v ${PWD}:/opt/test centos:7
```
```
docker volume ls
```

Volume nommé:
```
docker volume create volume-test
docker volume ls
```

```
docker run --rm -ti --name=centos-test -v volume-test:/opt/test centos:7
```
```
ls -la /opt/test/
```

```
docker run --rm -ti --name=centos-test -v volume-test:/var centos:7
```
```
ls /var/
```


```
docker run --rm -ti --name=centos-test -v volume-test:/opt/test centos:7
```
```
ls /opt/test
```

> **ATTENTION** : comme indiqué dans le livre vous devez ouvrir un second terminal en utilisant la commande ```vagrant ssh```.

Premier conteneur:
```
docker run --rm -ti -v volume-test:/opt/test centos:7
```
```
watch -n 1 cat /opt/test/fichier.txt
```

Second conteneur:
```
docker run --rm -ti -v volume-test:/opt/test centos:7
```
```
echo -e "Heure du moment $(date)" >> /opt/test/fichier.txt
```

### CONFIGURATION D’APPLICATION (§8.5)
```
docker run \
    --volume=/:/rootfs:ro \
    --volume=/var/run:/var/run:rw \
    --volume=/sys:/sys:ro \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --publish=8080:8080 \
    --detach=true \
    --name=cadvisor \
    --privileged=true \
    google/cadvisor:latest
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://spring.io/projects/spring-boot

https://www.dasblinkenlichten.com/using-cni-docker/

https://github.com/containernetworking/plugins

https://github.com/flannel-io/flannel#flannel

https://www.tigera.io/tigera-products/calico/

https://cilium.io/

https://www.netfilter.org/

https://kubedb.com/

https://galeracluster.com/library/documentation/docker.html

https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sqlserver-ver15&pivots=cs1-bash

https://www.mongodb.com/compatibility/docker

https://www.elastic.co/logstash/

https://cloud.spring.io/spring-cloud-config/ (redirigé vers https://cloud.spring.io/spring-cloud-config/reference/html/)

https://www.nagios.org/

https://prometheus.io/

https://newrelic.com/

