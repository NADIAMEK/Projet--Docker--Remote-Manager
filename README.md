
 Projet-Docker Remote Manager

1-Gestionnaire Docker à distance - Architecture client-serveur Java

[![Java](https://img.shields.io/badge/Java-17-orange.svg)](https://www.oracle.com/java/)
[![Docker](https://img.shields.io/badge/Docker-26.1.3-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Build](https://img.shields.io/badge/Build-Maven-yellow.svg)](https://maven.apache.org/)

2-  Table des Matières
-  Aperçu
-  Fonctionnalités
-  Architecture
-  Installation
-  Utilisation
-  Commandes Disponibles
-  Tests
-  Structure du Projet
-  Contribution
-  Licence


1.1 Aperçu

J-Docker Remote Manager est une application Java permettant de contrôler un moteur Docker distant via une interface ligne de commande.Ce projet implémente une architecture client-serveur robuste avec communication TCP et format JSON.

1.2 Contexte
Dans l'écosystème Cloud moderne, la gestion des infrastructures se fait via des API programmatiques. Ce projet répond au besoin de pilotage distant de conteneurs Docker via une console simplifiée.

1.3 Fonctionnalités

a- Implémentées
-  Connexion distante : Communication client-serveur TCP
-  Gestion des images : Liste, téléchargement depuis Docker Hub
-  Cycle de vie conteneurs : Création, démarrage, arrêt, suppression
-  Communication JSON : Protocole structuré et extensible
-  Multithreading : Serveur supportant plusieurs clients simultanés
-  Robustesse: Gestion des erreurs et déconnexions brusques
- Interface CLI : Expérience utilisateur intuitive avec emojis

1.4  En développement 
- Logs en temps réel (streaming)
- Interface web
- Sauvegarde/restauration

1.5 Architecture

```
┌─────────────────┐    TCP/5000    ┌─────────────────┐    TCP/2375    ┌─────────────────┐
│                 │ ──────────────▶ │                 │ ──────────────▶ │                 │
│   Client Java   │                 │  Serveur Java   │                 │    Docker       │
│   (Windows)     │ ◀────────────── │   (Proxy)       │ ◀────────────── │    (VM Ubuntu)  │
│                 │    JSON         │                 │    API REST     │                 │
└─────────────────┘                 └─────────────────┘                 └─────────────────┘
```

1.6 Composants
- Client: Interface CLI interactive (`DockerCLI.java`)
- Serveur : Daemon multithreadé (`DockerServer.java`)
- Proxy : Traduction commandes → API Docker (`DockerManager.java`)
- Communication : TCP sur port 5000, format JSON


1.7 Installation

   Prérequis
- Java 17+ ([Télécharger](https://www.oracle.com/java/technologies/javase-jdk17-downloads.html))
- Maven 3.6+ ([Télécharger](https://maven.apache.org/))
- Docker ([Guide d'installation](https://docs.docker.com/engine/install/))

1.8  Configuration Docker
```bash
# Sur la machine hébergeant Docker (VM Ubuntu)
sudo nano /etc/docker/daemon.json
```
Ajouter :
```json
{
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
```
Redémarrer Docker :
```bash
sudo systemctl restart docker
```

1.9 Installation du projet
```bash
# 1. Cloner le dépôt
git clone https://github.com/ton-username/j-docker-remote-manager.git
cd j-docker-remote-manager

# 2. Compiler avec Maven
mvn clean compile

# 3. Créer le JAR exécutable
mvn package
```


2. Utilisation

2.1  Lancer le serveur
```bash
# Option 1 : Directement avec Maven
mvn exec:java -Dexec.mainClass="com.jdocker.server.DockerServer"

# Option 2 : Avec le JAR généré
java -jar target/jdocker-server.jar
```

2.2  Lancer le client
```bash
# Option 1 : Directement avec Maven
mvn exec:java -Dexec.mainClass="com.jdocker.client.DockerCLI"

# Option 2 : Avec le JAR généré
java -jar target/jdocker-client.jar
```

3. Configuration réseau
Par défaut, le client se connecte à `localhost:5000`. Pour une connexion distante :

3.1 Modifier `DockerCLI.java`** :
```java
String serverAddress = "192.168.1.50"; // IP du serveur
```

3.2 Vérifier les pare-feux :
```powershell
# Sur Windows (PowerShell Admin)
New-NetFirewallRule -DisplayName "JDocker Port 5000" -Direction Inbound -LocalPort 5000 -Protocol TCP -Action Allow
```


4.  Commandes Disponibles

| Commande | Description | Exemple |
|----------|-------------|---------|
| `ping` | Test de connexion | `ping` |
| `version` | Version Docker | `version` |
| `images` | Liste des images | `images` |
| `pull <image>` | Télécharger une image | `pull nginx:alpine` |
| `run <image>` | Créer un conteneur | `run ubuntu:latest --name mon-conteneur` |
| `stop <id>` | Arrêter un conteneur | `stop abc123def456` |
| `rm <id>` | Supprimer un conteneur | `rm abc123def456` |
| `containers` | Liste des conteneurs | `containers` ou `ps` |
| `status` | État de la connexion | `status` |
| `help` | Afficher l'aide | `help` |
| `exit` | Quitter le programme | `exit` |

4.1  Exemples d'utilisation
```bash
# Scénario complet
jdocker> ping
 pong

jdocker> pull hello-world:latest
  Image hello-world:latest téléchargée avec succès

jdocker> run hello-world:latest --name test
  Conteneur créé et démarré...

jdocker> containers
 1 conteneurs:
ID           NOM                 IMAGE           STATUT       PORTS
abc123def456 test                hello-world:latest 🟢 RUNNING  0 port(s)

jdocker> stop abc123def456
  Conteneur abc123def456 arrêté

jdocker> rm abc123def456
   Conteneur abc123def456 supprimé
```

5.  Tests

5.1 Environnement de test
- Système hôte : Windows 11
- VM Docker : Ubuntu 20.04, Docker 26.1.3
- Réseau : Connexion TCP bridge
- Java : OpenJDK 17.0.2

5.2 Scénarios validés
```bash
# Test 1 : Connexion basique
  ping → pong
  version → Affichage version Docker

# Test 2 : Gestion images
   images → Liste formatée
   pull nginx:alpine → Téléchargement réussi

# Test 3 : Cycle de vie conteneurs
     run → Création conteneur
     containers → Affichage état
     stop → Arrêt propre
     rm → Suppression

# Test 4 : Robustesse
 Multi-clients simultanés (3 clients)
 Déconnexion brutale (Ctrl+C)
 Erreurs Docker gérées (image inexistante)
```

5.3  Lancer les tests
```bash
# Tests unitaires
mvn test

# Test manuel avec script
./scripts/test-manual.sh
```

6.  Structure du Projet

```
j-docker-remote-manager/
├── src/main/java/com/jdocker/
│   ├── client/
│   │   ├── DockerCLI.java          # Interface utilisateur CLI
│   │   └── CommandSender.java      # Gestion des connexions client
│   ├── server/
│   │   ├── DockerServer.java       # Serveur principal
│   │   ├── ClientHandler.java      # Gestion par client
│   │   ├── DockerService.java      # Service métier
│   │   └── DockerManager.java      # Communication avec Docker API
│   ├── models/
│   │   ├── Request.java            # Modèle requête
│   │   ├── Response.java           # Modèle réponse
│   │   ├── Container.java          # Modèle conteneur
│   │   └── Image.java              # Modèle image
│   └── utils/
│       ├── JsonUtil.java           # Utilitaires JSON
│       └── CommandParser.java      # Parser de commandes
├── src/test/java/                  # Tests unitaires
├── resources/
│   └── config.properties           # Configuration
├── scripts/
│   ├── start-server.sh             # Script démarrage serveur
│   ├── start-client.sh             # Script démarrage client
│   └── demo.sh                     # Script de démonstration
├── docs/
│   ├── architecture.png            # Diagramme d'architecture
│   ├── screenshots/                # Captures d'écran
│   └── presentation.pptx           # Présentation du projet
├── pom.xml                         # Configuration Maven
└── README.md                       # Ce fichier
```

7. Dépendances Maven
```xml
<dependencies>
    <dependency>
        <groupId>com.github.docker-java</groupId>
        <artifactId>docker-java</artifactId>
        <version>3.3.6</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.2</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

8.  Contribution

Les contributions sont les bienvenues ! Pour contribuer :

1. Fork le projet
2. Crée une branche (`git checkout -b feature/amazing-feature`)
3. Commit tes changements (`git commit -m 'Add amazing feature'`)
4. Push vers la branche (`git push origin feature/amazing-feature`)
5. Ouvre une Pull Request

9.  Standards de code
- Java: Google Java Style Guide
- Commits: Conventional Commits
- Documentation: JavaDoc pour les méthodes publiques

10. Fonctionnalités souhaitées
- [ ] Interface web (React/Spring Boot)
- [ ] Logs en streaming temps réel
- [ ] Monitoring des performances
- [ ] Support Docker Compose
- [ ] Authentification sécurisée

---

11.  Licence

Ce projet est sous licence ''MIT''. 

```
MIT License





















