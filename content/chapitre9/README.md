# Chapitre 9

> **IMPORTANT** : veuillez vous reporter au [chapitre 3](../chapitre3/README.md) pour installer le dépôt de code dans votre machine virtuelle vagrant. Les répertoires comme ```/application-exemple/``` se trouve dans le répertoire ```/vagrant/docker-exemples-edition3-code```. Le code pour ce chapitre se trouve donc dans ```/vagrant/docker-exemples-edition3-code/chapitre9```.

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [OPTION 1 : UN SEUL CONTENEUR, PLUSIEURS PROCESSUS (§9.2)](#option-1--un-seul-conteneur-plusieurs-processus-§92)
        - [Lancement du build (§9.2.3)](#lancement-du-build-§923)
        - [Lancement de l'application (§9.2.4)](#lancement-de-lapplication-§924)
        - [Gestion des volumes (§9.2.6)](#gestion-des-volumes-§926)
    - [OPTION 2 : APPLICATION MULTI-CONTENEURS (§9.3)](#option-2--application-multi-conteneurs-§93)
        - [Création de l’image (§9.3.1)](#création-de-limage-§931)
        - [Lancement de l’application (§9.3.2)](#lancement-de-lapplication-§932)
    - [OPTION 3 : ORCHESTRATION AVEC COMPOSE (§9.4)](#option-3--orchestration-avec-compose-§94)
        - [Introduction et premiers pas avec Compose (§9.4.1)](#introduction-et-premiers-pas-avec-compose-§941)
        - [Démarrage de l’application (§9.4.5)](#démarrage-de-lapplication-§945)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### OPTION 1 : UN SEUL CONTENEUR, PLUSIEURS PROCESSUS (§9.2)

Placez-vous dans le répertoire ```supervisor```.

#### Lancement du build (§9.2.3)
```
./build.sh
```

> **ATTENTION** : le processus de compilation est long (plusieurs minutes selon la vitesse de votre réseau et les performances de votre PC). Soyez patient.
> Vous trouverez un exemple de log complet du build dans le fichier [build-supervisor.log](build-supervisor.log).

#### Lancement de l'application (§9.2.4)
```
./run.sh
```
Visualisation des processus:
```
docker exec -it appex-supervisor ps axf
```

> **Note** :  vous avez oublié le mot de passe ? Il se trouve dans la page 176 du livre.

> **Note** : les fichiers CSV à charger sont accessibles depuis le répertoire dans lequel vous avez démarré Vagrant : ```docker-exemples-edition3-code/application-exemple/data```. Si vous avez suivi le processus d'installation du [chapitre 3](../chapitre3/README.md) celui-ci est accessible sur votre PC.

Pour réinitialiser l'application:
```
./clean.sh
```

#### Gestion des volumes (§9.2.6)

Option volumes nommés : placez-vous dans le répertoire ```supervisor```:
```
./run_volume_nommes.sh
```
Pour réinitialiser l'application:
```
./clean.sh
```
Option volumes hôtes : placez-vous dans le répertoire ```supervisor```:
```
./run_volume_hote.sh
```
```
docker exec -ti appex-supervisor cat /var/log/mariadb/mariadb.log
```
```
id vagrant
```
Pensez à décommenter l'instruction ```RUN useradd -u 1000 mysql``` dans le ```Dockerfile```.
```
docker cp appex-supervisor:/var/lib/mysql mysql
```
Pour réinitialiser l'application:
```
./clean.sh
```

### OPTION 2 : APPLICATION MULTI-CONTENEURS (§9.3)

Placez-vous dans le répertoire ```multi-conteneurs```.

#### Création de l’image (§9.3.1)

> Vous trouverez un exemple de log complet du build dans le fichier [build-multi-conteneurs.log](build-multi-conteneurs.log).

```
./build.sh
```
```
docker images
```

#### Lancement de l’application (§9.3.2)
```
./run.sh
```

Lo log de lancement devrait ressembler à:
```
[vagrant@localhost multi-conteneurs]$ ./run.sh
d5b8da89d1ddbc69fcf4df5d3400c83b545a094d82e113611f861aa4c4fde98b
appex-db-volume
appex-upload-files
Error response from daemon: No such container: appex-db
Error response from daemon: No such container: appex-front-end
Error response from daemon: No such container: appex-back-end
64031c9442526215d973d6fc9210aca89a43d4f2e5e3fed747fca7905d8e5b99
--------------
CREATE SCHEMA IF NOT EXISTS `insee_data` DEFAULT CHARACTER SET utf8
--------------

--------------
CREATE TABLE IF NOT EXISTS `insee_data`.`departement` (
  `code` VARCHAR(3) NOT NULL,
  `population` INT NOT NULL,
  `nombre_hotel` INT NOT NULL,
  `commune_plus_grande` VARCHAR(255) NOT NULL,
  `commune_plus_petite` VARCHAR(255) NOT NULL,
  `last_update` DATETIME NOT NULL,
  PRIMARY KEY (`code`))
ENGINE = InnoDB
--------------

8c6f409cc4c0c2c49219df87cf89aef3ce008ea25d548f75a086c85205c753e9
0d2854750aea893acf5115f894d556900817a2e4e05424500f5f28e92bff5ba0
Total reclaimed space: 0B
``` 

Pour réinitialiser l'application:
```
./clean.sh
```

### OPTION 3 : ORCHESTRATION AVEC COMPOSE (§9.4)

Placez-vous dans le répertoire ```compose```.

#### Introduction et premiers pas avec Compose (§9.4.1)

```
sudo curl -L https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
```
```
docker-compose --version
```

#### Démarrage de l’application (§9.4.5)

```
./run.sh
```

> Vous trouverez un exemple de log du lancement de compose [compose-up.log](compose-up.log).

```
docker-compose ps
```

Pour réinitialiser l'application:
```
./clean.sh
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://gradle.org/

http://supervisord.org/

https://fedoraproject.org/wiki/EPEL (redirigé vers https://docs.fedoraproject.org/en-US/epel/)

https://fr.wikipedia.org/wiki/Cron

https://yaml.org/

https://github.com/vishnubob/wait-for-it

