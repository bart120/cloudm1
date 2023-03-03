
# Projet Docker

Vous allez faire un projet conteneurs avec Docker.

# I- Sujet

## Applications 

L'objectif est la création, le déploiement, et l'éxécution et la mise à jour de différents micro-services REST et applications web dans un environnement conteneurisé.

## Ce qui est fournis

Pour ce projet, je jouerais le rôle du développeur. Vous pourrez donc me poser des questions sur le fonctionnement des apps, ce qu'elles ont besoin en terme de ressources et de liaisons. Si il le faut, je pourrais modifier le code des applications afin de les adapter à votre système (tant que ça reste de la configuration). Les applications sont disponibles sur le [git](https://github.com/bart120/cloudm1/tree/main/projet_docker/appscore) du cours.
Description des apps:
- web => application web qui consomme les services applicants.api et jobs.api
- applicants.api => service API REST fournissant des données d'une base de données.
- identity.api => service API REST fournissant une authentification
- jobs.api => service API REST fournissant des données d'une base de données.
- sql.data => base données SqlServer et données
- rabbitmq => fournis une communication entre les services API REST

## Les images

Vous devez créer les Dockerfile et image pour les apps Database, Applicants.Api, Identity.Api, Jobs.Api, Web.
Vous devrez créer les fichiers afin de générer des images de conteneur de chaque app. Les apps étant réalisées sur des technos différentes, il vous faudra bien préparer les images pour que les apps s'éxécutent sans problèmes (image de base, ouverture de port, mise en réseau..).
Les images des services devront par la suite être envoyées vers un repo d'images privé (dockerhub ou azure). Les images SqlServer et Web ne seront pas générée dans le repo et seront générées à la volet.

### Dockerfile Database:
Image de base utilisée: sqlserver2017
Les 3 fichiers sont fournis sont a copier dans le conteneur. Le fichier SqlCmdStartup.sh doit être éxécuter avant le démarrage du conteneur afin de préparer les données dans la base de données.
Il est recommandé d'utiliser un volume pour ce conteneur.
Le conteneur expose le port sqlserver 1433

### Dockerfile pour les services Applicants, Identity et Jobs
La contstruction se déroule en plusieurs étapes:
- Depuis l'image mcr.microsoft.com/dotnet/core/aspnet:2.1-stretch-slim qui vous nomerez "base", créer un répertoire /app et exposer les ports 80 et 443
- Depuis l'image mcr.microsoft.com/dotnet/core/sdk:2.1-stretch qui vous nomerez "build", créer un répertoire /src. Copier les dossiers Services/(dossier qui contient le fichier csproj) et Foundation/Events dans le répertoire /src.
- Exécuter la commande "dotnet restore {nom_du_projet.csproj}"
- Faites une copie intégrale
- Placez vous dans le dossier du projet (dossier qui contient le fichier csproj)
- Exécuter la commande "dotnet build "{nom_du_projet.csproj}" -c Release -o /app/build"
- Depuis l'image build vers une nouvelle image qui vous nomerez "publish", éxécuter la commande "dotnet publish "{nom_du_projet.csproj}" -c Release -o /app/publish"
- Depuis l'image base vers une nouvelle image qui vous nomerez "final", placez-vous dans le dossier /app, copier ce que vous avez créer dans l'image publish vers le dossier /app/publish de l'image final. 
- Exécuter le entrypoint suivant: ["dotnet", "{nom_du_projet.dll}"]

### Dockerfile pour Web

La constraction est quasi identique aux services précédants.
Web utilise la librairie "Foundation/Http" au lieu de "Foundation/Events".

## Les conteneurs

Voici les différents conteneurs (et dons services) que vous devez créer:

### user.data
Service basé sur l'image redis. Exposition du port 6379.
Ce service sera sur un réseau propre et ne pourra communiquer que avec les services applicants.api, jobs.api, identity.api.

### rabbitmq
Service basé sur l'image rabbitmq:3-management, il expose les ports 15672 et 5672.
Ce service sera sur un réseau propre et ne pourra communiquer que avec les services applicants.api, jobs.api, identity.api.

### sql.data
Service basé sur l'image Database que vous allez créée à la volée.
Ce service sera sur un réseau propre et ne pourra communiquer que avec les services applicants.api, jobs.api, identity.api.
Ce service disposera d'un volume afin de garantir l'intégritée des données.

### applicants.api
Service basé sur l'image créée plustôt. Expose son port 80 sur le 8081 de l'host.
Il doit pouvoir communiquer avec l'host et les services rabbimq et sql.data.
2 variables d'env: 
  - ConnectionString = Server={nom du service sql data};User=sa;Password=Pass@word;Database=dotnetgigs.applicants;
  - HostRabbitmq={nom du service rabibitmq}
Afin d'éviter les erreurs et redémarrage, le service rabbitmq devra être démarré et en statut healthy avant la création de ce service.

### jobs.api
Service basé sur l'image créée plustôt. Expose son port 80 sur le 8083 de l'host.
Il doit pouvoir communiquer avec l'host et les services rabbimq et sql.data.
2 variables d'env: 
  - ConnectionString = Server={nom du service sql data};User=sa;Password=Pass@word;Database=dotnetgigs.applicants;
  - HostRabbitmq={nom du service rabibitmq}
Afin d'éviter les erreurs et redémarrage, le service rabbitmq devra être démarré et en statut healthy avant la création de ce service.

### identity.api
Service basé sur l'image créée plustôt. Expose son port 80 sur le 8084 de l'host.
Il doit pouvoir communiquer avec l'host et les services rabbimq et redis.
2 variables d'env: 
  - RedisHost={nom du service redis}:6379
  - HostRabbitmq={nom du service rabibitmq}
Afin d'éviter les erreurs et redémarrage, le service rabbitmq devra être démarré et en statut healthy avant la création de ce service.

### web
Service basé sur l'image créée plustôt. Expose son port 80 sur le 80 (ou 8080) de l'host.
Il n'a pas de communication direct avec les autres conteneurs sauf les 3 api. Mais ceux-ci doivent être en exécution obligatoirement.
Afin d'éviter les erreurs et redémarrage, les service api devront être démarrés avant la création de ce service.

### sécurité
Supprimer les ouvertures de ports non obligatoire.

# Groupes et fonctionnement

Le projet est a faire seul.

Les étudiants sont encouragés (mais pas obligés) à mettre en place un système de contrôle des sources de type Git ou équivalent, afin d'affecter et partager efficacement les fichiers de codes et autres composants numériques du projet (fichiers sources, descripteurs de déploiement, documents de recherche, cas d'utilisation, etc.).

# Rendu

Le rendu se fera le vendredi 10 mars 2023 maxi.
Toute absence à la soutenance entrainera un 0 (ZERO).
Vous devrez présenter chacun votre tour votre travail réalisé et le résultat obtenu.