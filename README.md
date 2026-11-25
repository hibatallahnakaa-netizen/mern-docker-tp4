# TP4 - Docker et Docker Compose (application MERN)

Introduction au DevOps - ISG Sousse - 3 LIG - Enseignante : Zaineb Sakhrawi

## Liens
- GitHub : https://github.com/hibatallahnakaa-netizen/mern-docker-tp4
- Docker Hub (serveur) : https://hub.docker.com/r/hibatallahnakaa/mern-server
- Docker Hub (client) : https://hub.docker.com/r/hibatallahnakaa/mern-client

## Fichiers
- `Server/Dockerfile` : image de l'API Express (node:lts-alpine, port 9000, `npm start`)
- `Client/Dockerfile` : image du client React/Vite (build puis `serve` sur le port 3000)
- `docker-compose.yml` : services mongodb, server, client sur le reseau `mern-network` (bridge)
- `Jenkinsfile` : pipeline Checkout, Build, Deploy, Smoke Test, Push Docker Hub
- `Captures/` : captures d'ecran (pipeline Jenkins, images, conteneurs)

## Commandes utilisees et leur role

| Commande | Role |
|---|---|
| `git clone https://github.com/wiem-kb/MyAppReact.git` | Recuperer le projet de depart (dossiers Client et Server) |
| `docker build -t mern-server ./Server` | Construire l'image du serveur a partir de son Dockerfile |
| `docker build -t mern-client ./Client` | Construire l'image du client (compile React puis le sert) |
| `docker network create mern-network` | Creer un reseau pour que les conteneurs communiquent par leur nom |
| `docker run -d --name mongodb --network mern-network mongo` | Lancer MongoDB en arriere-plan sur le reseau |
| `docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/myapp mern-server` | Lancer l'API, exposer le port 9000, fournir l'URL MongoDB |
| `docker run -d --name client --network mern-network -p 3000:3000 mern-client` | Lancer le client et l'exposer sur le port 3000 |
| `docker ps` | Verifier que les conteneurs tournent |
| `docker images` | Lister les images construites |
| `docker rm -f client server mongodb` | Supprimer les conteneurs manuels avant d'utiliser Compose |
| `docker compose up --build` | Construire et lancer les 3 services avec Docker Compose |
| `docker compose ps` | Verifier l'etat des services Compose |
| `docker compose down` | Arreter et supprimer les services et le reseau |
| `docker login` | Se connecter a Docker Hub |
| `docker compose push` | Publier les images client et serveur sur Docker Hub |
| `git remote set-url origin <url>` | Pointer le projet vers mon propre depot GitHub |
| `git add .` / `git commit -m "..."` / `git push -u origin main` | Enregistrer et publier le projet sur GitHub |

## Explication du Docker Compose
- `mongodb` : image officielle `mongo`, sur le reseau `mern-network`.
- `server` : construit depuis `./Server`, port 9000, variable `MONGO_URI=mongodb://mongodb:27017/myapp` (le nom du service sert de hostname), depend de `mongodb`.
- `client` : construit depuis `./Client`, port 3000, depend de `server`.
- Le reseau `mern-network` utilise le driver `bridge`.

## Lancer le projet
    docker compose up --build
- Client : http://localhost:3000
- API : http://localhost:9000

## Pipeline Jenkins
1. Checkout : recuperation du code depuis GitHub
2. Build images : docker build du serveur et du client
3. Deploy : reseau mern-network puis docker run de mongodb, server et client
4. Smoke Test : verification que le client repond (HTTP 200)
5. Push (Docker Hub) : connexion avec dockerhub-creds puis docker push des deux images

Note : le service Jenkins sous Windows ne dispose pas de la commande `docker compose`. Le pipeline utilise donc les commandes `docker build` et `docker run` equivalentes. Le fichier `docker-compose.yml` est teste en local avec `docker compose up --build`.

