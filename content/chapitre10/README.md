# Chapitre 10

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [UN ENVIRONNEMENT DE BUILD LUI-MÊME DOCKERISÉ (§10.2)](#un-environnement-de-build-lui-même-dockerisé-§102)
        - [Préparation des images de l’environnement de build (§10.2.2)](#préparation-des-images-de-lenvironnement-de-build-§1022)
    - [INSTALLATION DES OUTILS ET CHARGEMENT DU CODE SOURCE (§10.3)](#installation-des-outils-et-chargement-du-code-source-§103)
        - [Configuration de GitLab (§10.3.1)](#configuration-de-gitlab-§1031)
        - [Chargement initial du code source (§10.3.2)](#chargement-initial-du-code-source-§1032)
        - [Configuration de Jenkins (§10.3.3)](#configuration-de-jenkins-§1033)
    - [IMAGE ET JOB DE BUILD (§10.4)](#image-et-job-de-build-§104)
        - [Configuration du job dans Jenkins (10.4.3)](#configuration-du-job-dans-jenkins-1043)
    - [LANCEMENT AUTOMATIQUE (§10.5)](#lancement-automatique-§105)
        - [Configuration de Jenkins et GitLab (§10.5.1)](#configuration-de-jenkins-et-gitlab-§1051)
        - [Exemple de modification et déploiement (§10.5.2)](#exemple-de-modification-et-déploiement-§1052)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### UN ENVIRONNEMENT DE BUILD LUI-MÊME DOCKERISÉ (§10.2)

#### Préparation des images de l’environnement de build (§10.2.2)

Placez-vous dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre10```.

```
./run.sh
```
> Vous trouverez un exemple de log complet de l'exécution de ```run.sh``` dans le fichier [run.log](run.log).

### INSTALLATION DES OUTILS ET CHARGEMENT DU CODE SOURCE (§10.3)

#### Configuration de GitLab (§10.3.1)

> **ATTENTION** : n'oubliez pas de créer les **2** projets dans GitLab ```appex-front-end``` et ```appex-front-end-configuration```.

#### Chargement initial du code source (§10.3.2)

Placez-vous dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre10/environnement_de_developpement/code_source```.
```
./push_code.sh
```

Placez-vous dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre10/environnement_de_developpement/fichiers_configuration```.
```
./push_fichiers.sh
```

#### Configuration de Jenkins (§10.3.3)

```
docker exec -ti jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

> **IMPORTANT** : Dans le livre nous présentons une interface graphique pour Jenkins en français. Par défaut l'outil choisit la langue du navigateur. Si votre navigateur fonctionne en anglais, l'interface sera donc en anglais.

> **ERRATUM** : Sur la page 237 on indique que le script à installer est "PostBuildScript" alors qu'il s'agit du script "Post build task". Vous noterez que la capture d'écran est à l'inverse correcte.

> **ATTENTION** : Lorsque vous installez le plugin si vous choisissez de redémarrer jenkins, le conteneur va s'arrêter. Si c'est le cas (vous le verrez avec la commande ```docker ps```, il vous suffit de le redémarrer avec la commande ```docker start jenkins```).


### IMAGE ET JOB DE BUILD (§10.4)

#### Configuration du job dans Jenkins (10.4.3)

```
/opt/build/scripts/start-build.sh ${BUILD_TAG}
/opt/build/scripts/build-java.sh ${BUILD_TAG}
/opt/build/scripts/build-configure.sh ${BUILD_TAG} "DEV"
/opt/build/scripts/build-front-end-image.sh ${BUILD_TAG}
```
> **ERRATUM** : Dans le menu déroulant "Actions à la suite du build" c'est "Post build task" qu'il faut sélectionner. Il n'y a pas de menu "Script" (ou du moins celui-ci ne correspond pas à ce que nous devons utiliser).

> **ERRATUM** : La capture d'écran de la page 240 est redimensionnée de sorte qu'elle est difficile à lire. Elle est de plus incorrecte car le champ "Log text" doit rester vide de sorte que le script s'exécute après chaque build quoi qu'il arrive. Voici la version en taille originale ci-dessous:

![Figure-10.13](Figure-10.13.png)

```
/opt/build/scripts/end-build.sh ${BUILD_TAG}
```

### LANCEMENT AUTOMATIQUE (§10.5)

#### Configuration de Jenkins et GitLab (§10.5.1)

Désactivation de la sécurité sur les API dans Jenkins:
> **ATTENTION** : pensez à noter la valeur du TOKEN_SECRET générée par Jenkins

Création du WebHook dans GitLab:
```
http://admin:<<token d'authentification>>@jenkins:8080/job/CI_BUILD_DEV/build?token=123456789
```
> Pensez à remplacer le ```<<token d'authentification>>``` par la valeur du TOKEN_SECRET que vous avez noté précédemment.

#### Exemple de modification et déploiement (§10.5.2)

> Pour déclencher un build toute modification du fichier ```src/main/resources/templates/home.html``` fonctionne. La figure 10.20 montre simplement qu'on a remplacé "Bienvenue" par "Bienvenue à tous".

```
docker inspect --format '{{ json .ContainerConfig.Labels}}' appex-multi/front-end
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://www.jenkins.io/

https://about.gitlab.com/

https://github.com/jpetazzo/dind

https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/

https://www.redhat.com/sysadmin/building-buildah

https://search.maven.org/

