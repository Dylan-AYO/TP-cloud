# TP1 Prise en main de Docker

## I. Prise en main
### 1. Lancer des conteneurs
#### Manipulation du conteneur
* pour lister les conteneurs j'utile la commande "sudo docker ps".
![](https://i.imgur.com/mhHpGF7.png)

* pour montrer l'arborescence de processus j'utilise la commande "ps faux"

dans le conteneur on a ce résultat : 
![](https://i.imgur.com/chy2B7B.png)

sur la VM on a plus de processus.
![](https://i.imgur.com/ZFdYSbf.png)

les cartes réseaux ne sont pas les mêmes que sur la VM. Avec la commande "ip addr" on a une carte réseaux et une adresse ip différente de la VM.

la commande "cat /etc/passwd" permet de lister tout les utilisateurs du conteneur. on peut voir que l'utilisateur dylan n'est pas sur le conteneur.

il n'y a pas de partitions montées sur le conteneur. on peut vérifier cela avec la commande "fdisk -l ". La commande "fdisk -l" permet de lister les partitions.

* pour détruire le conteneur il faut d'abord l'arrêter avec "sudo docker stop dazzling_driscoll" puis la supprimer avec "sudo docker rm dazzling_driscoll".

#### Lancer un conteneur NGINX

pour lancer un conteneur nginx on utilise la commande suivante "sudo docker run -p 80:80 -d nginx"
on utilise ensuite l'adresse ip de ma VM dans un navigateur pour vister le service web.
![](https://i.imgur.com/YGf9dJ7.png)

### 2. Gestion d'images
#### Gestion d'images

pour récupérer une image de Apache en version 2.2 on utilise la commande "sudo docker pull cytopia/apache-2.2".

pour lancer un conteneur avec l'image apache "sudo docker run -p 80:80 -d cytopia/apache-2.2"

#### Création d'image
* pour créer une image qui qui lance un serveur web python on utilise un Dockerfile.

Dockerfile:

FROM alpine
RUN apk add --no-cache python3
EXPOSE 80:80
WORKDIR /home/test
COPY ./test/hello /home/test
CMD python3 -m http.server 8888

* pour lancer le conteneur et accéder au serveur web on utilise la commande suivante
"sudo docker run -p 7777:8080 -d alpine/python"

* pour créer un volume on utilise la commande suivante
"sudo docker run -v /home/dylan/test:/home/test -d alpine/python"

### 3. Manipulation du démon docker

#### 1. Modifier la configuration du démon Docker

pour modifier le socket utiliser par docker on utilise la commande
"sudo dockerd -H unix:///var/run/docker.sock -H tcp://192.168.20.129"


#### 2 Docker-compose

* Ecrire un docker-compose-v1.yml qui permet de :

version: "3.9"  # optional since v1.27.0
services:
  AlpinePython:
    image: alpine/python
    ports:
           - '80:80'
           
* ajouter un conteneur Nginx

version: "3.9"  # optional since v1.27.0
services:
  AlpinePython:
    image: alpine/python
    networks:
      - net

  reverse-proxy:
    image: nginx
    volumes:
      - ./nginx:/etc/nginx
    ports:
      - 80:443
    networks:
      - net

networks:
       net:


