# Dockerfile
# Layer 1: Image node 
FROM node:17-alpine 

# Layer 2: Définir le dossier route du container
WORKDIR /app 

# Layer 3: copy les package.json dans le dossier du container
COPY package.json . 

# Layer 4: Installation des dépendances
RUN npm install 

# Layer 5: Copier tous les fichiers depuis le dossier actuel vers le container
COPY . . 

# Layer 6: Définition du port
EXPOSE 3000 

# Layer 7: Run npm start
CMD ["npm", "start"]