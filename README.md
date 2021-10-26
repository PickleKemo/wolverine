# Wolverine Project
- This Project containes three docker compose that allows to create four images as follows:
1. Traefik allows our services to pass through a proxy and loadbalance them and expose them to the external with a tls resolver and secure them
2. SonarQube is our code scanner for any vulnarability and so on
3. PostgreSQL to store data of SonarQube
4. Harbor is our registery for our docker images
## How to set up this project
1. First you need to clone the project on your local machine
---
git clone https://logan.cefim-formation.org/root/tp_wolverine.git
---
2- Have walkthorugh the folders with e.g
```
cd /Harbor
```
3- As you can see every folder has a compose file 

