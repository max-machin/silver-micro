# Dockerfile
# Layer 1: Image node:10
FROM node:10

# Layer 2: Création du dossier de l'app
WORKDIR /usr/src/app

# Layer 3: Installer les dépendances.
COPY package*.json ./
RUN npm install

# Layer 4: Copier le contenu de l'app
COPY . .

# Layer 5: Port à écouter par le container
EXPOSE 8080

# Layer 6: Run de la commande node server.js
CMD [ "node", "server.js" ]