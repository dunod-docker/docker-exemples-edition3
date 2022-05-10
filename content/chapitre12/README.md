# Chapitre 12

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [PRISE EN MAIN (§12.1)](#prise-en-main-§121)
        - [IMPORTANT : Installation des exemples (§12.2.1)](#important--installation-des-exemples-§1221)
        - [Kubectl :  la CLI de Kubernetes (§12.2.2)](#kubectl---la-cli-de-kubernetes-§1222)
        - [Un exemple simple (§12.2.3)](#un-exemple-simple-§1223)
        - [Un descripteur Kubernetes (§12.2.4)](#un-descripteur-kubernetes-§1224)
    - [DECOUVERTE DES FONCTIONNALITES (§12.3)](#decouverte-des-fonctionnalites-§123)
        - [Ajout d’un état : écriture sur un volume host (§12.3.1)](#ajout-dun-état--écriture-sur-un-volume-host-§1231)
            - [Lancement du pod et description](#lancement-du-pod-et-description)
            - [Ajoutons un volume](#ajoutons-un-volume)
        - [Influencer le scheduler Kubernetes : les labels (§12.3.2)](#influencer-le-scheduler-kubernetes--les-labels-§1232)
            - [Définition d’un label](#définition-dun-label)
            - [Définition des exigences pour un pod](#définition-des-exigences-pour-un-pod)
        - [Persistent volumes : la persistance gérée par Kubernetes (§12.3.3)](#persistent-volumes--la-persistance-gérée-par-kubernetes-§1233)
            - [Création du secret Kubernetes pour l’authentification](#création-du-secret-kubernetes-pour-lauthentification)
        - [Création des objets Kubernetes et du Pod](#création-des-objets-kubernetes-et-du-pod)
        - [Haute disponibilité basique et avancée (§12.3.4)](#haute-disponibilité-basique-et-avancée-§1234)
            - [Résilience automatique](#résilience-automatique)
            - [Réplicas : gestion de plusieurs pods au sein d’un même déploiement](#réplicas--gestion-de-plusieurs-pods-au-sein-dun-même-déploiement)
            - [Services : répartition de charge](#services--répartition-de-charge)
            - [Auto-scaling : montée et répartition de charge automatique](#auto-scaling--montée-et-répartition-de-charge-automatique)
    - [DÉPLOIEMENT DE L’APPLICATION EXEMPLE (§12.4)](#déploiement-de-lapplication-exemple-§124)
        - [Nettoyage (§12.4.1)](#nettoyage-§1241)
        - [Build et publication des images (§12.4.2)](#build-et-publication-des-images-§1242)
        - [Création des volumes (§12.4.5)](#création-des-volumes-§1245)
        - [Configuration et déploiement des composants (§12.4.6)](#configuration-et-déploiement-des-composants-§1246)
            - [Pre-initialisation de la base de données](#pre-initialisation-de-la-base-de-données)
            - [Lancement des autres composants et test](#lancement-des-autres-composants-et-test)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

> **IMPORTANT** : placez-vous à nouveau dans le répertoire que vous avez créé au [chapitre 3](../chapitre3/README.md), celui dans lequel vous avez installé les applications exemples. Lancez la commande ```vagrant up``` puis ```vagrant ssh``` pour vous connecter dans la machine virtuelle. Vous allez travailler, dans la VM, dans le répertoire ```/vagrant/docker-exemples-edition3-code/chapitre12```.

### PRISE EN MAIN (§12.1)

#### IMPORTANT : Installation des exemples (§12.2.1)

Attention : il est important de bien remplir le fichier ```configuration.env``` puis de rendre exécutable le fichier ```setup.sh``` à l'aide de la commande suivante:
```
chmod u+x setup.sh
``` 

Pour les paramètres STORAGE_ACCOUNT_NAME et STORAGE_ACCOUNT_KEY, veuillez-vous reporter aux pages 274-275 qui expliquent la création d'un Azure Storage Account.
Celle-ci nécessite comme l'usage d'Azure Kubernetes Service un accès à un compte Azure et au portail https://portal.azure.com.

Pour le paramètre DOCKER_HUB_ACCOUNT, entrez le "login" de votre compte DockerHub (https://hub.docker.com/).
Le mot de passe vous sera demandez au lancement des scripts qui effectuent des "push" vers votre registry.
Attention, toutes ces images sont publiques par défaut, ne publiez rien qui soit secret.

Pour installer ```kubectl``` pour Azure, veuillez vous reporter aux liens suivants:
* https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest : décrit l'installation de la ligne de commande az
* https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest :
  * explique comment installer ```kubectl``` à l'aide de la commande:
  ```
  az aks install-cli
  ```
  * explique les instructions pour obtenir les "credentials" (paramètres d'authentification pour AKS) à l'aide de la commande:
  ```
  az aks get-credentials
  ```

#### Kubectl :  la CLI de Kubernetes (§12.2.2)

```
kubectl version
```

```
kubectl get componentstatuses
```

```
kubectl get nodes
```

#### Un exemple simple (§12.2.3)

Dans ```chapitre12/quelleheure/prise-en-main```:
```
./build.sh
./deploy.sh
```

```
kubectl get pods
```

Connection au pod:
```
kubectl port-forward quelleheure 8090:80
```

Visualiser les logs:
```
kubectl logs -f  quelleheure
```

#### Un descripteur Kubernetes (§12.2.4)

Effacement du pod:
```
kubectl delete pod quelleheure
```

### DECOUVERTE DES FONCTIONNALITES (§12.3)

#### Ajout d’un état : écriture sur un volume host (§12.3.1)

##### Lancement du pod et description

Dans ```chapitre12/quelleheure/log-sans-volume```:
```
./build.sh
./deploy.sh
```

```
kubectl describe pods quelleheure
```

```
kubectl port-forward quelleheure 8090:80
```

Redémarrage:
```
kubectl delete pods quelleheure
kubectl apply -f quelleheure-log-sans-volume.yaml
```

##### Ajoutons un volume

Dans ```chapitre12/quelleheure/log-volume-host```:
```
./build.sh
./deploy.sh
```
Attention, le fichier ```deploy.sh``` contient l'instruction utilisant le modificateur ```--force```.

```
kubectl port-forward quelleheure 8090:80
```

#### Influencer le scheduler Kubernetes : les labels (§12.3.2)

##### Définition d’un label

Affichage de la liste des nodes:
```
kubectl get nodes
```

Marquage de l'un de nodes:
```
kubectl label nodes <<nom de l'un de vos nodes>> log=ok
```

```
kubectl get nodes --show-labels
```

##### Définition des exigences pour un pod

Dans ```chapitre12/quelleheure/labels```:
```
./deploy.sh
```

```
kubectl get pods -o wide
```

Se connecter pour effacer le fichier de log:
```
kubectl exec -ti quelleheure /bin/bash
```

#### Persistent volumes : la persistance gérée par Kubernetes (§12.3.3)

##### Création du secret Kubernetes pour l’authentification

Création du volume persistant.
Dans ```chapitre12/quelleheure/persistent-volume```:
```
./create-azure-secret.sh
```

#### Création des objets Kubernetes et du Pod

Dans ```chapitre12/quelleheure/persistent-volume```:
```
./deploy-pv.sh
./deploy-pvc.sh
./deploy-pod.sh
```

```
kubectl get pv
```

```
kubectl describe  pods quelleheure
```

#### Haute disponibilité basique et avancée (§12.3.4)

##### Résilience automatique

Connection au pod pour "détruire" un processus apache:
```
kubectl exec -ti quelleheure /bin/bash
```

```
kubectl get pods -o wide
```

IMPORTANT :
```
kubectl delete pod quelleheure
```

##### Réplicas : gestion de plusieurs pods au sein d’un même déploiement

IMPORTANT :
```
kubectl delete pod quelleheure
```

Dans ```chapitre12/quelleheure/deployment```:
```
./deploy-deployment.sh
```

##### Services : répartition de charge

Dans ```chapitre12/quelleheure/deployment```:
```
./deploy-service.sh
```

```
kubectl get pods -o wide
```

```
kubectl get services quelleheure-service --watch
```

##### Auto-scaling : montée et répartition de charge automatique

```
kubectl autoscale deployment quelleheure-deployment --cpu-percent=50 --min=1 --max=5
```

### DÉPLOIEMENT DE L’APPLICATION EXEMPLE (§12.4)

#### Nettoyage (§12.4.1)

Dans ```chapitre12/application-exemple```:
```
kubectl delete service quelleheure-service
kubectl delete deployments --all
kubectl delete pvc shared-volume-claim
kubectl delete pyv shared-volume
```

#### Build et publication des images (§12.4.2)

Dans ```chapitre12/application-exemple```:
```
./build.sh
./push-imnages.sh
```

#### Création des volumes (§12.4.5)

Dans ```chapitre12/application-exemple\deploy```:
```
kubectl apply -f creation-volume.yaml
```

```
kubectl get pv
kubectl get pvc
```

#### Configuration et déploiement des composants (§12.4.6)

##### Pre-initialisation de la base de données

Dans ```chapitre12/application-exemple\deploy```:
```
./init-db.sh
```

Après l'exécution de l'initialisation de la base de données, nous n'avons plus besoin du pod:
```
kubectl delete pod init-db
```

##### Lancement des autres composants et test

Dans ```chapitre12/application-exemple\deploy```:
```
kubectl apply -f creation-database-service.yaml
kubectl apply -f creation-front-end.yaml
kubectl apply -f creation-back-end.yaml
```

```
kubectl get service
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://minikube.sigs.k8s.io/docs/start/

https://kind.sigs.k8s.io/docs/user/quick-start/

https://kubernetes.io/docs/setup/independent/install-kubeadm/ (redirigé vers https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

https://azure.microsoft.com/fr-fr/services/kubernetes-service/

https://kubernetes.io/docs/reference/kubectl/

https://kubernetes.io/docs/concepts/configuration/overview/

https://code.visualstudio.com/

https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes

https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv

https://severalnines.com/database-blog/running-galera-cluster-kubernetes

https://helm.sh/

https://artifacthub.io/



