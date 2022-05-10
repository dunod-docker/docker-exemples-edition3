# Chapitre 6

<!-- TOC depthFrom:2 -->

- [1- Commandes du chapitre](#1--commandes-du-chapitre)
    - [LES MODÈLES D’INSTRUCTION (§6.1)](#les-modèles-dinstruction-§61)
        - [Terminal ou exécution ? (§6.1.2)](#terminal-ou-exécution--§612)
    - [LES INSTRUCTIONS D’UN DOCKERFILE (§6.2)](#les-instructions-dun-dockerfile-§62)
        - [FROM (§6.2.1)](#from-§621)
        - [CMD (§6.2.3)](#cmd-§623)
        - [ENTRYPOINT (§6.2.4)](#entrypoint-§624)
        - [EXPOSE (§6.2.5)](#expose-§625)
        - [ADD (§6.2.6)](#add-§626)
        - [COPY (§6.2.7)](#copy-§627)
        - [VOLUME (§6.2.8)](#volume-§628)
- [2- Liens (dans l'ordre d'apparition dans le chapitre)](#2--liens-dans-lordre-dapparition-dans-le-chapitre)

<!-- /TOC -->

## 1- Commandes du chapitre

### LES MODÈLES D’INSTRUCTION (§6.1)

#### Terminal ou exécution ? (§6.1.2)
```
/bin/sh -c "ping localhost"
```

### LES INSTRUCTIONS D’UN DOCKERFILE (§6.2)

#### FROM (§6.2.1)
```
FROM centos:7
CMD echo "Hello world"
FROM centos:7
CMD echo "Bonjour à tous"
```

```
docker build .
```

#### CMD (§6.2.3)

Exemple 1:
```
FROM centos:7
ENTRYPOINT ["/bin/ping","-c","5"]
CMD ["localhost"]
```
```
docker build -t my-ping .
```
```
docker run my-ping
```

Exemple 2:
```
FROM centos:7
CMD ping localhost
```
```
docker build -t my-ping-cmd .
```
```
docker run my-ping-cmd ls
```

Exemple 3:
```
FROM centos:7
ENTRYPOINT ping localhost
```
```
docker build -t my-ping-entrypoint .
```
```
docker run --entrypoint ls my-ping-entrypoint
```

Surcharge de my-ping:
```
docker run my-ping google.com
```

#### ENTRYPOINT (§6.2.4)
Format exécution:
```
FROM centos:7
ENTRYPOINT ["ping","google.com"]
```
```
docker build -t ping-google-exec .
```
```
docker run --rm --name test ping-google-exec
```

Format terminal:
```
FROM centos:7
ENTRYPOINT ping google.com
```
```
docker build -t ping-google-shell .
```
```
docker run --rm --name test ping-google-shell
```

#### EXPOSE (§6.2.5)
```
FROM centos:7
RUN yum update -y && yum install -y \
openssh-server \
passwd
RUN mkdir /var/run/sshd
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''
RUN useradd user
RUN echo -e "pass\npass" | (passwd --stdin user)
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```
```
docker build -t ssh .
```

Mappage automatique:
```
docker run -P -d ssh
```

Mappage manuel:
```
docker run -d -p 22 ssh
```
```
docker run -d -p 35022:22 ssh
```
```
docker inspect --format='{{json .ContainerConfig.ExposedPorts}}' ssh {"22/tcp":{}}
```
```
ssh user@localhost -p 35022
```

#### ADD (§6.2.6)
```
FROM centos:7
RUN pwd > /tmp/initialPath
RUN mkdir output
RUN cd output
RUN pwd > /tmp/pathAfterOutput
ADD test1 ./
ADD ["test2" , "/output/"]
CMD ls /output
```
```
docker build -t add .
```
```
docker run --rm add
```
```
docker run --rm add ls /
```
```
docker run --rm add cat /tmp/initialPath
```
```
docker run --rm add cat /tmp/pathAfterOutput
```

Quand une facilité de l'instruction ADD devient un inconvénient:
```
FROM centos:7
ADD wordpress-4.4.1-fr_FR.tar.gz /tmp/
CMD ls /tmp
```
```
docker build -t untar .
```
```
docker run --rm untar
```
```
docker run --rm untar ls /tmp/wordpress
```

#### COPY (§6.2.7)
Dockerfile multicopy1:
```
FROM centos:7
COPY test1.tar.gz /tmp/
RUN tar xzf /tmp/test1.tar.gz
COPY test2.tar.gz /tmp/
RUN tar xzf /tmp/test2.tar.gz
```

Dockerfile multicopy2:
```
FROM centos:7
COPY test1.tar.gz test2.tar.gz /tmp/
RUN tar xzf /tmp/test1.tar.gz
RUN tar xzf /tmp/test2.tar.gz
```

```
docker build -t multicopy1 .
docker build -t multicopy2 .
```

#### VOLUME (§6.2.8)
Avec volume:
```
FROM centos:7
VOLUME /tmp/data
CMD ping localhost
```
```
docker build -t volume .
docker run -d --name volume-conteneur volume
```
```
docker inspect --format='{{json .Mounts}}' volume-conteneur
```
```
docker exec volume-conteneur ls /tmp/data
```
```
docker exec volume-conteneur /bin/sh -c 'echo "Hello" > /tmp/data/helloTest'
```
```
docker exec volume-conteneur ls /tmp/data
```
```
docker stop volume-conteneur
```
```
docker rm volume-conteneur
```
```
docker run -d -v /var/home/vagrant/data:/tmp/data --name volume-conteneur volume
```
```
docker exec volume-conteneur /bin/sh -c 'echo "Hello" > /tmp/data/helloTest'
```
```
sudo ls /var/home/vagrant/data
```
> ```sudo``` n'est en fait pas nécessaire dans l'exemple ci-dessus

```
docker stop volume-conteneur
docker rm volume-conteneur
```

Sans volume:
```
FROM centos:7
# VOLUME /tmp/data
CMD ping localhost
```
```
docker build -t volume .
docker run -d -v /var/home/vagrant/data:/tmp/data --name volume-conteneur volume
docker exec volume-conteneur /bin/sh -c 'echo "Hello" > /tmp/data/helloTest'
```
```
docker inspect --format='{{json .Mounts}}' volume-conteneur
```
```
ls /var/home/vagrant/data/
```

## 2- Liens (dans l'ordre d'apparition dans le chapitre)

https://docs.docker.com/develop/develop-images/multistage-build/

