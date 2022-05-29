# Contenu complémentaire

<!-- TOC depthFrom:2 -->

- [Rancher desktop : l'alternative open-source à Docker Desktop](#rancher-desktop--lalternative-open-source-à-docker-desktop)
- [Buildah : scripter la construction d'images](#buildah--scripter-la-construction-dimages)
    - [L'option ARG](#loption-arg)
    - [L'option Buildah](#loption-buildah)
    - [Usage de buildah](#usage-de-buildah)
- [Kaniko - construire des images dans un cluster Kubernetes](#kaniko---construire-des-images-dans-un-cluster-kubernetes)

<!-- /TOC -->

## Rancher desktop : l'alternative open-source à Docker Desktop

Rancher est un acteur connu du monde des conteneurs. Fondé en 2014, Rancher est l'éditeur de l'une des offres (encore indépendante) de cluster Kubernetes on-premise les plus utilisée (probablement après OpenShift).

En Octobre 2020, sentant probablement des changements arriver après l'annonce de la vente de la suite entreprise de Docker Inc à Mirantis (cf. [chapitre 2](../chapitre2/)), Rancher commençait à travailler à une proposition alternative à Docker Desktop : Rancher Desktop.

Cette solution est étonnamment arrivée à maturité avec l'annonce en Août 2021, par Docker Inc., que docker Desktop deviendrait payant le 31 Janvier 2022.

Aujourd'hui, on peut dire que Rancher Desktop (https://rancherdesktop.io) est un clone complet, Open-Source et gratuit à Docker Desktop. Il offre même la possibilité de choisir son moteur de conteneur : soit Dockerd ou soit Containerd (avec [nerdctl](https://github.com/containerd/nerdctl), qui reprend la ligne de commande Docker).


## Buildah : scripter la construction d'images

Nous évoquons dans le livre, à plusieurs reprises, l'outil (Buildah)[https://buildah.io/] le compagnon de (Podman)[https://podman.io/], tous deux développés sous la houlette de RedHat.

Le principe de Buildah est de permettre de "scripter" la création d'image OCI standard.

Imaginons que nous souhaitions construire une image ```nginx``` (le serveur web).
Imaginons encore que nous souhaitions créer dynamiquement, en fonction d'un paramètre obtenu au moment du build de l'image, alternativement deux variantes:
* avec comme home page le contenu d'un fichier ```index-option1.html```
* avec comme home page le contenu d'un fichier ```index-option2.html```

Voici le code pour ```index-option1.html```:
```
<!DOCTYPE html>
<html>
<head>
<title>Option 1</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to home page option 1</h1>
</body>
</html>
```

Voici le code pour ```index-option2.html```:
```
<!DOCTYPE html>
<html>
<head>
<title>Option 2</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to home page option 1</h1>
</body>
</html>
```

Rien de très flamboyant mais l'idée semble claire : comment construire une image avec deux variantes dynamiquement.

### L'option ARG

L'option "Dockerfile compatible" est d'utiliser ARG (nous présentons cette instruction en détail au [chapitre 7](../content/chapitre7/README.md)).

Voici à quoi ressemblerait le Dockerfile:
```
FROM nginx
ARG option
ADD index-option${option}.html /usr/share/nginx/html/index.html
```

L'instruction de construction d'image pour l'option1 serait donc:
```
podman build -t myimage --build-arg option=1 -f Dockerfile-args
```
> remplacer ```option=1``` par ```option=2``` changerait l'image résultante.

On pourrait ensuite vérifier le fonctionnement de l'image en produisant un conteneur:
```
podman run --rm -d -p8000:80 --name webserver myimage
```

Le navigateur ouvert sur http://localhost:8000 vous révèlerait l'option embarquée dans l'image. 

Dans cet exemple, rien de bien compliqué.
Mais si la variante souhaitée était plus complexe que le simple choix d'un fichier (un calcul par exemple, un programme complet), celui-ci devrait s'exécuter au moment de la création d'image et devrait être décrit dans le Dockerfile. Les étapes intermédiaires du processus complexe pourraient générer des résidus (fichiers ou autre) qui seraient alors "embarqués" dans l'image.

Aussi puissant que puisse être la syntaxe Dockerfile, il ne s'agit pas d'un langage de programmation universel.

Buildah se propose justement d'offrir la possibilité de créer une image grace à des instructions en ligne de commande.

### L'option Buildah

Le script suivant permet d'obtenir un résultat équivalent:
```
#!/bin/bash
buildah from nginx
buildah copy nginx-working-container index-option$1.html /usr/share/nginx/html/index.html
buildah commit nginx-working-container myimage
```

Buildah fonctionne en créant un conteneur qu'on modifie pour finir en un commit qu'on sauve sous la forme d'une image (c'est une approche similaire à celle que nous présentons le [chapitre 4](../content/chapitre4/README.md) page 85).

Si ce code était placé dans un fichier ```use_buildah.sh```, on l'utiliserait de la manière suivante:
```
./use_buildah.sh 2
```
> remplacer ```1``` par ```2``` changerait l'image résultante.

Dans notre cas la variante est simple (un COPY), mais nous pourrions envisager de sélectionner dynamiquement (avec des if, des boucles, etc.) tout type d'instruction docker grâce à l'option ```onbuild```:
```
buildah config --onbuild="RUN touch /bar" nginx-working-container
```

Buildah s'apparente donc à un Dockerfile entièrement programmable.

### Usage de buildah

En réalité la production d'images OCI par l'intermédiaire de scripts n'est pas un besoin très courant. La plupart des développeurs se contenteront d'un Dockerfile souvent simple.

La valeur de Buildah s'exprime plutôt dans les environnements CI/CD conteneurisés.
Buildah est en effet l'option de choix si on souhaite créer une image dans un conteneur.

Buildah n'est pas un moteur de conteneur mais simplement d'un outil de création d'image.
On a donc pas besoin de recourir à un client Docker complet communiquant avec l'hôte via une socket unix comme nous le faisons dans le [chapitre 10](../content/chapitre10/README.md) page 230.
Buildah peut exécuter un Dockerfile classique sans besoin de communiquer avec l'hôte.
Il s'agit donc d'une option plus sûre en termes de sécurité.

> Buildah aura néanmoins besoin de pousser l'image résultante vers un registry.
```
buildah push localhost/myimage monregistry:5000/myimage
```

## Kaniko - construire des images dans un cluster Kubernetes

[Kaniko](https://github.com/GoogleContainerTools/kaniko) est un outil du même type que Buildah. Il permet de construire des images OCI à l'intérieur d'un conteneur.

La différence réside dans le fait qu'on ne peut utiliser que l'image officielle Kaniko (```gcr.io/kaniko-project/executor```). Kaniko est un outil qui prend un Dockerfile et pousse une image vers un registry, à l'inverse de buildah qui offre une interface en ligne de commande pour construire une image en se passant du Dockerfile si nécessaire.

Kaniko a été initié en 2018 (première release) par des ingénieurs de Google mais n'est pas officiellement supporté (c'est indiqué de manière proéminente sur le README GitHub).
