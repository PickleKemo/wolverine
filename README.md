# Wolverine Project
- This Project containes three docker compose that allows to create four images as follows:
1. Traefik allows our services to pass through a Reverse proxy and loadbalance them and expose them to the external with a tls resolver and secure them
2. SonarQube is our code scanner for any vulnarability and so on
3. PostgreSQL to store data of SonarQube
4. Harbor is our registery for our docker images
## How to set up this project
1. First you need to clone the project on your local machine
```
git clone https://logan.cefim-formation.org/root/tp_wolverine.git
```
2. Have walkthorugh the folders for e.g
```
cd /tp_wolverine
ls 
README.md  harbor  sonarqube  traefik
cd sonarqube
ls
docker-compose.yml
```
3. As you can see every folder has a docker compose file 
```
cd sonarqube
ls
docker-compose.yml
```
### Mont our containers
- We will start with Traefik since it is our Reverse proxy
#### First let's understand our config and our docker-compose of Traefik Reverse proxy
***traefik.yml***
```
api:
  dashboard: true  # this on to allow us to access the dashboard in our nav
  insecure: false  # to allow our services to go through proxy only if they are secured

# Entry Points configuration
# redirect our http to https
entryPoints:
  web:
    address: :80
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
# use the tls  resolver to Certificates our services
  websecure:
    address: :443
    http:
      tls:
        certResolver: production

# Certificates configuration
# ---
# TODO: Custmoize your Cert Resolvers and Domain settings
#
certificatesResolvers:
# LET'S ENCRYPT:
# ---
#staging use only for test
  staging:
    acme:
      email: example@hotmail.com  # TODO: Change this to your email
      storage: /ssl-certs/acme.json
      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web
#production use only to publish 
  production:
    acme:
      email: example-aomran@hotmail.com  # TODO: Change this to your email
      storage: /ssl-certs/acme.json
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web


providers:
# DOCKER:
# expose only the services that are enable to pass through traefik
  docker:
    exposedByDefault: false  # Default is true

```
***docker-compose.yml***
```
# define the networks and it's name
networks: 
  traefik_proxy:
    name: traefik_proxy

services:
  traefik:
    image: "traefik:v2.3" # pull the image
    container_name: "traefik" # name your container
    ports: 
      - "80:80"
      - "443:443"
    volumes: 
      - /etc/traefik:/etc/traefik # our config file
      - traefik-ssl-certs:/ssl-certs # store our certificate
      - /var/run/docker.sock:/var/run/docker.sock:ro # sockets on read only
      - /var/log/traefik/:/var/log/traefik/:rw # read and write logs
    networks:
      - traefik_proxy # call the network
    labels:
      - traefik.enable=true # enable the provider
      - traefik.http.routers.traefik.entrypoints=websecure # use the secured entryPoint
      - traefik.http.routers.traefik.rule=Host(`DNS`) # change DNS by your own 
      - traefik.http.routers.traefik.service=api@internal # use the service
```
1. we will have to move the file that is located in our config folder to ***/etc/Traefik***, and to do so we will have to creat a the folder in ***/etc/***
```
cd /etc
mkdir traefik
cd /traefik/config
mv traefik.yml /etc/traefik/
```
2. We go back to our traefik file where our docker compose is 
```
docker-compose up -d
```
#### Second we do SonarQube
***docker-compose.yml***
```
---
version: "3.2"

services:
  db:
    image: postgres:12.1 # pull DB image
    environment:
      - POSTGRES_USER=sonar # change me
      - POSTGRES_PASSWORD=mypass # change me
      - POSTGRES_DB=sonarqube
    networks:
      - traefik_proxy  # we use our existing proxy that we setted up previously
    volumes:
      - sonarqube_db:/var/lib/postgresql/data 

  sonarqube:
    image: sonarqube:7.7-community # pull SonarQube image
    environment:
      - sonar.jdbc.username=sonar # change me
      - sonar.jdbc.password=mypass # change me
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonarqube
    networks:
      - traefik_proxy # we use our existing proxy that we setted up previously
    volumes: 
      - sonarqube_conf:/opt/sonarqube/conf  # configuration
      - sonarqube_extensions:/opt/sonarqube/extensions # extensions
      - sonarqube_logs:/opt/sonarqube/logs # logs
      - sonarqube_data:/opt/sonarqube/data # data
      - sonarqube_bundled-plugins:/opt/sonarqube/bundled-plugins # bundled plugins
    labels:
      - "traefik.enable=true" # enable it to be accessed via the internal
      - "traefik.http.routers.sonarqube.rule=Host(`DNS`)" # change me
      - "traefik.http.routers.sonarqube.entrypoints=websecure" # we secure it

volumes: # define volumes
  postgresql_data:
  sonarqube_bundled-plugins:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_db:
  sonarqube_extensions:
  sonarqube_logs:
networks: # define network
  traefik_proxy:
    name: traefik_proxy
```
