# Chapitre 7

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [VARIABLES D’ENVIRONNEMENT ET CONTENEURS : ENV (§7.1)](#variables-denvironnement-et-conteneurs--env-§71)
    - [META-INFORMATION ER IMAGES : LABEL (§7.2)](#meta-information-er-images--label-§72)
    - [PARAMÉTRER LE BUILD D’UNE IMAGE (§7.3)](#paramétrer-le-build-dune-image-§73)
        - [WORKDIR (§7.3.1)](#workdir-§731)
        - [ARG (§7.3.2)](#arg-§732)
        - [ONBUILD (§7.3.3)](#onbuild-§733)
    - [MODIFIER LE CONTEXTE SYSTÈME AU COURS DU BUILD (§7.4)](#modifier-le-contexte-système-au-cours-du-build-§74)
        - [SHELL (§7.4.1)](#shell-§741)
        - [USER (§7.4.2)](#user-§742)
    - [AUTO-GUÉRISON (SELF HEALING) (§7.5)](#auto-guérison-self-healing-§75)
        - [--restart (§7.5.1)](#--restart-§751)
        - [HEALTHCHECK (§7.5.2)](#healthcheck-§752)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### VARIABLES D’ENVIRONNEMENT ET CONTENEURS : ENV (§7.1)
```
FROM centos:7
ENV myName="James Bond" myJob=Agent\ secret
CMD echo $myName
```
```
docker build -t env .
```
```
docker run --rm env
```
```
docker run --rm --env myName="Jason Bourne" env
```
```
docker run --rm --env myName="Jason Bourne" --env myJob="CIA" env
```
```
docker run --rm env /bin/sh -c 'echo "$myJob"'
```

Inspection:
> **ERRATA** : Dans le livre l'option "-d" n'est pas présente dans l'instruction ci-dessous. Elle n'est pas absolument nécessaire mais permet d'éviter d'avoir à ouvrir un autre terminal (avec ```vagrant ssh```). Cette option permet au conteneur de fonctionner en mode démon (en tâche de fond).

```
docker run -d --rm --name test env ping localhost
```
```
docker inspect --format='{{json .Config.Env}}' test
```

> Attention : n'oubliez pas de stopper et de supprimer le conteneur ```test```.

### META-INFORMATION ER IMAGES : LABEL (§7.2)
```
FROM centos:7
LABEL app.name="Mon application" \
"app.version"="1.0" \
app.description="Voici la description de \"Mon application\" \
sur plusieurs lignes."
CMD echo "fin"
```
```
docker build -t label .
docker run --name test label
```
```
docker inspect --format='{{json .Config.Labels}}' test
```

### PARAMÉTRER LE BUILD D’UNE IMAGE (§7.3)

#### WORKDIR (§7.3.1)
```
FROM centos:7
WORKDIR /tmp
WORKDIR test
CMD pwd
```
> Les deux commandes ci-dessous ne sont pas présentes dans le livre mais elles permettent de visualiser l'explication du livre
```
docker build -t test-workdir .
```
```
docker run --rm test-workdir
```

Image workdirsource:
```
FROM centos:7
WORKDIR /var
RUN pwd > /tmp/cheminCourant
CMD cat /tmp/cheminCourant
```
> Les deux commandes ci-dessous ne sont pas présentes dans le livre mais elles permettent de visualiser l'explication du livre
```
docker build -t workdirsource .
```
```
docker run --rm workdirsource
```
```
docker run --rm workdirsource pwd
```

Extension:
```
FROM workdirsource
CMD pwd
```
```
docker build -t workdirsource-ext .
```
```
docker run --rm workdirsource-ext
```

#### ARG (§7.3.2)
Variable vide:
```
FROM centos:7
ARG var1
ARG var2="ma valeur"
RUN echo $var1 > /tmp/var
RUN echo $var2 >> /tmp/var
CMD echo $var1
```
```
docker build --build-arg var1=valeur1 -t arg .
```
```
docker run --rm arg
```

Modification de CMD:
```
FROM centos:7
ARG var1
ARG var2="ma valeur"
RUN echo $var1 > /tmp/var
RUN echo $var2 >> /tmp/var
CMD cat /tmp/var
```
```
docker build --build-arg var1=valeur1 -t arg .
```
```
docker run --rm arg
```

Surchase de var2:
```
docker build --build-arg var1=valeur1 --build-arg var2="une autre valeur" -t arg .
```
```
docker run --rm arg
```

Conflit de variable:
```
FROM centos:7
ARG var=argument
ENV var variable
RUN echo $var > /tmp/var
CMD cat /tmp/var
```
```
docker build -t arg .
```
```
docker run --rm arg
```

Inversement des assignements:
```
FROM centos:7
ENV var variable
ARG var=argument
RUN echo $var > /tmp/var
CMD cat /tmp/var
```
```
docker build -t arg .
```
```
docker run --rm arg
```

Cumul des instructions ARG et ENV:
```
FROM centos:7
ARG version
ENV version ${version:-1.0}
RUN echo $version > /tmp/version
CMD cat /tmp/version
```
```
docker build -t arg .
```
> **ERRATUM** : dans le libre il manque la commande docker pour cette instruction
```
docker run --rm arg
```
```
docker build --build-arg version=1.2 -t arg .
```
```
docker run --rm arg
```

#### ONBUILD (§7.3.3)
```
FROM centos:7
ONBUILD ARG fichier="app.py"
ONBUILD COPY $fichier /app/app.py
ONBUILD RUN python -m py_compile /app/app.py
ENTRYPOINT ["python", "/app/app.pyc"]
```
```
docker build -t python-app:1.0-onbuild .
```

Fichier python ```x2.py```:
```
import sys
x=int(sys.argv[1])
resultat=x*2
print(repr(x)+" * 2 = " +repr(resultat))
```

> Si vous souhaitez tester ce code python sur l'hôte, la bonne commande est la suivante.
```
python2 x2.py 5
```
> En effet l'installation par défaut de python est la version 2 et il est nécessaire de suffixer la commande python avec la version.

Extension:
```
FROM python-app:1.0-onbuild
CMD ["0"]
```
```
docker build --build-arg fichier=x2.py -t python-x2 .
```
```
docker run --rm python-x2 17
```

Inspection:
```
docker inspect --format='{{json .ContainerConfig.OnBuild}}' python-app:1.0-onbuild
```

### MODIFIER LE CONTEXTE SYSTÈME AU COURS DU BUILD (§7.4)

#### SHELL (§7.4.1)
> La commande présente dans le livre ne peut pas être exécutée avec la configuration Linux/vagrant. Il faut disposer d'un environnement Docker pour windows. Si vous tentez de construire une image sous Linux, voici le message d'erreur que vous verrez:
```
docker build -t win .

Sending build context to Docker daemon  288.1MB
Step 1/1 : FROM microsoft/windowsservercore
pull access denied for microsoft/windowsservercore, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

#### USER (§7.4.2)
Fichier initial:
```
FROM centos:7
RUN chown root:root /bin/top && \
chmod 774 /bin/top
RUN groupadd -r mygroup && \
useradd -r -g mygroup myuser
USER myuser
CMD /bin/top -b
```
```
docker build -t user .
```
```
docker run --rm --name user-conteneur user
```

Modification avec user:
```
FROM centos:7
RUN chown root:root /bin/top && \
chmod 774 /bin/top
RUN groupadd -r mygroup && \
useradd -r -g mygroup myuser
RUN chown myuser:mygroup /bin/top
USER myuser
CMD /bin/top -b
```
```
docker build -t user .
```
```
docker run --rm --name user-conteneur user
```
```
docker inspect --format='{{json .Config.User}}' user-conteneur "myuser"
```

Une approche alternative:
```
FROM centos:7
RUN yum update -y && yum install -y sudo
RUN chown root:root /bin/top && \
chmod 774 /bin/top
RUN groupadd -r mygroup && \
useradd -r -g mygroup myuser
RUN echo '%mygroup ALL=NOPASSWD: /bin/top' >> /etc/sudoers
USER myuser
CMD sudo /bin/top -b
```
```
docker build -t user .
```
```
docker run --rm --name user-conteneur user
```

### AUTO-GUÉRISON (SELF HEALING) (§7.5)

#### --restart (§7.5.1)
```
docker run -d --name=sepuku centos:7 /bin/sh -c "sleep 5"
```
```
docker ps -a
```
```
docker run -d --restart=always --name=sepuku centos:7 /bin/sh -c "sleep 5"
```

#### HEALTHCHECK (§7.5.2)

> **ERRATA** : La version de Flask utilisée dans le livre ne fonctionne plus avec les versions récentes de python 3. Nous proposons donc de changer (dans le Dockerfile) pour la version 2.1.2.

Fichier ```app.py```:
```
from flask import Flask
from flask import request
from datetime import datetime

app = Flask(__name__)
calls=[]

@app.route('/')
def index():
    return 'Call count : <ul><li>{}</li></ul>'.format('</li><li>'.join(calls))

@app.route('/health')
def health():
    global calls
    dt = datetime.now()
    calls.append("from {} at {}".format(request.remote_addr,dt.strftime("%H:%M:%S")))
    return 'OK'

if __name__ == '__main__':
    app.run(host="0.0.0.0")
```

Dockerfile de l'image pour exécuter le code python:
```
FROM python:3.7.0
RUN pip install flask==2.1.2
COPY app.py app.py
HEALTHCHECK --interval=10s \
CMD curl -f http://localhost:5000/health || exit 1
ENTRYPOINT python app.py
```
```
docker build -t health-flask .
```
```
docker run -d --rm -p 8000:5000 --name=health-flask health-flask
```

```
docker events --since 10m --until 10s --filter "event=health_status:unhealthy" --format '{{.ID}}'
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

http://flask.pocoo.org/ (redirigé vers https://flask.palletsprojects.com/en/2.0.x/)

