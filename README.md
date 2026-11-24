# TP4 : Docker et Docker Compose (application MERN)

**Nom :** Sarra Hajjar - Groupe 03B
**Cours :** Introduction au DevOps 

## Objectif
Conteneuriser une application MERN (MongoDB, Express, React, Node.js) avec Docker, puis l'orchestrer avec Docker Compose.

## 1. Cloner le projet
```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```
Le projet contient deux dossiers : `client` (React) et `server` (Express).

## 2. Dockerfile du serveur (`server/Dockerfile`)
```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
ENV MONGO_URI=mongodb://mongodb:27017/mydb
CMD ["npm", "start"]
```
- `FROM` : image de base Node.js légère (Alpine).
- `WORKDIR` : répertoire de travail dans le conteneur.
- `COPY package*.json` puis `RUN npm install` : installation des dépendances.
- `COPY . .` : copie du reste du code.
- `EXPOSE 9000` : port de l'API.
- `ENV MONGO_URI` : URL de connexion à MongoDB.
- `CMD` : démarrage de l'application.

Construction de l'image :
```bash
docker build -t mern-server ./server
```

## 3. Dockerfile du client (`client/Dockerfile`)
```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```
Le projet utilise Vite : le build est généré dans le dossier `dist`. Le serveur léger `serve` sert les fichiers statiques sur le port 3000.

Construction de l'image :
```bash
docker build -t mern-client ./client
```

## 4. Réseau Docker
```bash
docker network create mern-network
```

## 5. Exécution des conteneurs
```bash
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
docker ps
```
Application accessible sur `http://localhost:3000` et API sur `http://localhost:9000`.

## 6. Docker Compose (`docker-compose.yml`)
```yaml
services:
  mongodb:
    image: mongo
    container_name: mongodb
    networks:
      - mern-network

  server:
    build: ./server
    container_name: server
    ports:
      - "9000:9000"
    environment:
      - MONGO_URI=mongodb://mongodb:27017/mydb
    depends_on:
      - mongodb
    networks:
      - mern-network

  client:
    build: ./client
    container_name: client
    ports:
      - "3000:3000"
    depends_on:
      - server
    networks:
      - mern-network

networks:
  mern-network:
    driver: bridge
```
- Les services communiquent via leur nom (le serveur accède à MongoDB avec le hostname `mongodb`).
- `depends_on` définit l'ordre de démarrage : mongodb, puis server, puis client.
- Le réseau `mern-network` utilise le driver `bridge`.

Lancement :
```bash
docker compose up --build
```
Arrêt :
```bash
docker compose down
```

## Captures d'écran
- `docker ps` : les 3 conteneurs (mongodb, server, client) sont `Up`.
- `http://localhost:3000` : application React.
- `http://localhost:9000` : API Express.

## Conclusion
L'application MERN est conteneurisée avec Docker et orchestrée avec Docker Compose en une seule commande.