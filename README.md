
# TP3 : Déploiement de l'application Spring Boot dans Kubernetes

Ce document décrit les étapes pour déployer l'application **stock-ms** sur Minikube en utilisant deux approches :

- **Méthode en ligne de commande** dans le namespace `jee`
- **Méthode déclarative** via des fichiers YAML dans le namespace `devops`

---

## Partie 1 : Déploiement en Ligne de Commande (Namespace `jee`)

### Prérequis
- Minikube installé et configuré avec le driver Docker.
- Kubectl configuré.

### Étapes

1. **Démarrer Minikube**  
   Lancez Minikube avec le driver Docker :
   ```bash
   minikube start --driver=docker
   ```

2. **Créer le Namespace**  
   Créez le namespace `jee` :
   ```bash
   kubectl create namespace jee
   ```

3. **Configurer l'Environnement Docker de Minikube**  
   Configurez votre terminal pour utiliser le démon Docker de Minikube :
   ```bash
   minikube docker-env | Invoke-Expression
   ```

4. **Créer le Déploiement**  
   Déployez l'application `stock-ms` dans le namespace `jee` :
   ```bash
   kubectl create deployment stock-ms --image=nabousoum/stock-ms:3.0 -n jee
   ```

5. **Exposer le Déploiement**  
   Créez un service de type NodePort pour exposer l'application sur le port 8080 :
   ```bash
   kubectl expose deployment stock-ms --type=NodePort --port=8080 --target-port=8080 -n jee
   ```

6. **Accéder au Service**  
   Récupérez l'URL du service et testez le déploiement :
   ```bash
   minikube service stock-ms -n jee --url
   ```

---

## Partie 2 : Déploiement Déclaratif (Namespace `devops`)

### Étapes

1. **Construire et Pousser l'Image Docker**

   - **Compiler l'application**  
     Utilisez Maven pour nettoyer et packager l'application :
     ```bash
     mvn clean package
     ```

   - **Construire l'image Docker**  
     Construisez l'image Docker localement :
     ```bash
     docker build -t stock-ms .
     ```

   - **Se connecter à Docker Hub**  
     Connectez-vous à Docker Hub :
     ```bash
     docker login
     ```

   - **Tagger l'image Docker**  
     Ajoutez un tag à l'image locale pour le dépôt Docker Hub :
     ```bash
     docker tag stock-ms:latest nabousoum/stock-ms:3.0
     ```

   - **Pousser l'image sur Docker Hub**  
     Envoyez l'image vers Docker Hub :
     ```bash
     docker push nabousoum/stock-ms:3.0
     ```

2. **Déployer via les Fichiers YAML**

   - **Appliquer le Namespace**  
     Créez le namespace `devops` à partir d’un fichier YAML :
     ```bash
     kubectl apply -f namespace-devops.yaml
     ```

   - **Déployer l'Application**  
     Appliquez la configuration du déploiement pour `stock-ms` :
     ```bash
     kubectl apply -f stock-ms-deployment.yaml
     ```

   - **Exposer le Service**  
     Appliquez la configuration du service pour exposer l'application :
     ```bash
     kubectl apply -f stock-ms-service.yaml
     ```

3. **Accéder au Service**  
   Récupérez l'URL du service et testez le déploiement :
   ```bash
   minikube service stock-ms -n devops --url
   ```

---

## Récapitulatif des Commandes

### Partie 1 (Namespace `jee`)
```bash
minikube start --driver=docker
kubectl create namespace jee
minikube docker-env | Invoke-Expression
kubectl create deployment stock-ms --image=nabousoum/stock-ms:3.0 -n jee
kubectl expose deployment stock-ms --type=NodePort --port=8080 --target-port=8080 -n jee
minikube service stock-ms -n jee --url
```

### Partie 2 (Namespace `devops`)
```bash
mvn clean package
docker build -t stock-ms .
docker login
docker tag stock-ms:latest nabousoum/stock-ms:3.0
docker push nabousoum/stock-ms:3.0
kubectl apply -f namespace-devops.yaml
kubectl apply -f stock-ms-deployment.yaml
kubectl apply -f stock-ms-service.yaml
minikube service stock-ms -n devops --url
```
