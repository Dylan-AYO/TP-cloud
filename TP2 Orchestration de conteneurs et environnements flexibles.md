# TP2 : Orchestration de conteneurs et environnements flexibles

## I. Lab setup


## II. Mise en place de Docker Swarm

### 1. Setup

Création du cluster Swarm
sur le node1 on utilise la commande "docker swarm init --advertise-addr 192.168.56.2" pour créer le cluster.
avec "docker swarm join" on obtient la commande suivante 
"docker swarm join --token SWMTKN-1-184xxpoff335xxe3b3vyjs10ou1w7azdkem6r55wbh184gpcyz-20kdwl4rvx66hest4uubka107 192.168.56.2:2377"
on utilise cette commande sur le node2 et node3 pour les joindre au cluster en tant que manager.

### 2. Une première application orchestrée

#### "swarmiser" l'application Python du TP1

* Docker-compose.yml:

version: "3.9"
services:

  certificate:
    image: alpine
    restart: "no"
    entrypoint: /cert/generate.sh
    volumes:
      - ./nginx/cert/:/cert/


  web-python:
    image: web_python
    restart: on-failure
    ports:
     - "8888"
    networks:
      web-net:
        aliases:
          - python.tp1.b3
          - python

  nginx:
    container_name: nginx
    restart: on-failure
    image: nginx
    volumes:
      - ./nginx/conf/nginx.conf:/etc/nginx/conf.d/web.tp1.b3.conf
      - ./nginx/cert/:/certs/
    ports:
      - "443:443"
    networks:
      web-net:
        aliases:
          - web.tp1.b3
          - web
    depends_on:
      - web-python
      - certificate

networks:
  web-net:
  
* l'image build sur les nodes du cluster est une alpine avec python d'installé.

on utilise ensuite la commande "docker stack deploy -c /data/docker-compose/docker-compose.yml web-python".

#### explorer l'application et les fonctionnalités de Swarm

## III. Construction de l'écosystème

### 1. Registre

#### Déployer un registre Docker simple

on déploie le node1 avec la commande "docker stack deploy -c /data/tp2-III/docker-compose.yml registry". Les répertoires du docker-compose doivent être créés avant.

Sur les nodes on ajoute l'entrée suivante au fichier hosts "192.168.56.2    registry.b3" pour que l'IP de node1 pointe vers registry.b3.

on créer en suite le daemon.json dans /etc/docker si il n'y est pas. On ajoute les lignes suivante: 
{
    "insecure-registries" : [ "registry.b3:5000" ]
}

on peut maintenant relancer le daemon docker, pousser et récupérer des images depuis notre registre.

#### Tester le registre et utiliser le registre

on renome l'image pour le registre avec la commande suivante
docker tag web_python registry.b3:5000/web_python/web_python:test

on pousse en suite l'image avec "docker push registry.b3:5000/web_python/web_python:test"

dans le docker-compose.yml on change le nom de l'image
web-python:
    image: registry.b3:5000/web_python/web_python
    restart: on-failure
    ports:
     - "8888"
    networks:
      web-net:
        aliases:
          - python.tp1.b3
          - python
          
### 2. Centraliser l'accès aux services

#### Déployer une stack Traefik

pour créer une stack traefik on utilise la commande "docker network create --driver overlay traefik".
on peut vérifier sur les nodes que le network est bien créé
"docker network ls".

#### Déployer une stack Traefik

dans le docker-compose.yml on définie le nom de domaine par "dylan.ayo.com".
sur notre client, on modifie le fichier hosts pour que le nom de domaine pointe vers une adresse ip d'un node.
Sur un navigateur "http://dylan.ayo.com" nous renvoie sur le dashboard de Traefik "https://dylan.ayo.com/dashboard/#/"

#### Passer l'application Web Python derrière Traefik

on utilise encore une fois le docker-compose.yml de l'application Web Python dans lequel on ajoute des labels.
docker-compose.yml:
version: "3.9"
services:

  certificate:
    image: alpine
    restart: "no"
    entrypoint: /cert/generate.sh
    volumes:
      - ./nginx/cert/:/cert/


  web-python:
    image: python:3-alpine
    restart: on-failure
    networks:
      traefik:
        aliases:
          - python.tp1.b3
          - python
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.python-webapp.rule=Host(`python.webapp.tp3`)"
        - "traefik.http.routers.python-webapp.entrypoints=web"
        - "traefik.http.services.python-webapp.loadbalancer.server.port=8888"
        - "traefik.http.routers.python-webapp-tls.rule=Host(`python.webapp.tp3`)"
        - "traefik.http.routers.python-webapp-tls.entrypoints=webtls"
        - "traefik.http.routers.python-webapp-tls.tls.certresolver=letsencryptresolver"

  nginx:
    container_name: nginx
    restart: on-failure
    image: nginx
    volumes:
      - ./nginx/conf/nginx.conf:/etc/nginx/conf.d/web.tp1.b3.conf
      - ./nginx/cert/:/certs/
    networks:
      traefik:
        aliases:
          - web.tp1.b3
          - web
    depends_on:
      - web-python
      - certificate
      
networks:
  traefik:
  
### 3. Swarm Management WebUI