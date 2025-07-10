# Installer Jenkins + Squash TM avec Docker Compose

Ce dépôt contient une configuration **Docker Compose** permettant de déployer rapidement un environnement d’intégration continue complet avec :

- **Jenkins** (avec support Docker-in-Docker)
- **Squash TM** (Community Edition)
- **Squash Orchestrator** (édition DEVOPS Community)
- **Mailhog** (optionnel, pour test de mails)

Cette configuration permet d’exécuter un pipeline Jenkins qui :

- Clone un projet GitHub
- Exécute des tests unitaires (ex : JUnit via Maven)
- Publie les résultats dans Jenkins
- Envoie les résultats dans Squash TM via l’Orchestrateur

⚠️ Cet environnement est destiné à des démonstrations ou TP uniquement. Il n’est **pas adapté à un usage en production**.

---

## Pré-requis

- Docker et Docker Compose installés (Docker Desktop recommandé)

---

## Étapes d’installation

### Étape 1 – Récupération du dépôt

Clonez ce dépôt ou téléchargez son contenu sur votre machine :

```bash
git clone <URL_DU_DEPOT>
cd <nom_du_dossier>
```

### Étape 2 – Préparation des plugins Squash

Avant de démarrer les conteneurs :

1. Créez un dossier `squash_plugins` à la racine du projet.
2. Téléchargez depuis le site officiel de [Squashtest](https://tm-fr.doc.squashtest.com/v7/downloads.html) les plugins suivants :
   - **Result Publisher**
   - **Squash AUTOM (community)**
   - **Test Plan Retriever**
3. Extrayez les fichiers `.jar` et placez-les dans `./squash_plugins/`.

> Ces plugins sont nécessaires pour permettre à Squash TM de communiquer avec l’Orchestrateur.

### Étape 3 – Construction de l’image Jenkins

Dans le dossier contenant le `Dockerfile`, exécutez :

```bash
docker build -t my-jenkins .
```

Cela prépare une image Jenkins avec le client Docker installé (pour permettre les étapes en conteneur).

### Étape 4 – Lancement des services

Démarrez tous les services avec :

```bash
docker compose up -d
```

Les services suivants sont lancés :

- Jenkins (accessible sur [http://localhost:8080](http://localhost:8080))
- Squash TM (accessible sur [http://localhost:8090/squash](http://localhost:8090/squash))
- Squash Orchestrator (ports 7774, 7775)
- Mailhog (optionnel : [http://localhost:8025](http://localhost:8025))

### Étape 5 – Configuration de Jenkins

1. Accédez à [http://localhost:8080](http://localhost:8080)
2. Récupérez le mot de passe administrateur via :

   ```bash
   docker exec my-jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

3. Terminez l’assistant Jenkins.
4. Installez le **plugin Squash DEVOPS** manuellement via :
   - *Manage Jenkins > Manage Plugins > Advanced > Upload plugin*
   - Téléchargez le fichier `.hpi` depuis : <https://www.squashtest.org/download/> (version community)

5. Allez dans *Manage Jenkins > Configure System*, ajoutez une configuration pour **Squash Orchestrator** (utilisez les URLs internes du docker-compose et le token JWT affiché dans les logs du conteneur `squash-orchestrator`).

### Étape 6 – Configuration de Squash TM

1. Connectez-vous à [http://localhost:8090/squash](http://localhost:8090/squash) (`admin` / `admin`)
2. Créez un projet, un cas de test automatisé et une itération.
3. Associez un serveur d'automatisation (Orchestrateur) dans *Administration > Servers > Test Automation Servers*.
4. Copiez l’UUID de l’itération créée pour l’utiliser dans le pipeline Jenkins.

---

## Exemple de pipeline Jenkins

Un pipeline de test typique fait les étapes suivantes :

- Clone d’un projet GitHub (ex : [LableOrg/java-maven-junit-helloworld](https://github.com/LableOrg/java-maven-junit-helloworld))
- Exécution des tests avec Maven
- Publication des résultats dans Jenkins
- Envoi des résultats à Squash TM via l’orchestrateur

Le fichier `Jenkinsfile` correspondant est fourni dans ce dépôt.

---

## Gestion des conteneurs

### Arrêter les services

```bash
docker compose down
```

### Reprendre plus tard

```bash
docker compose up -d
```

### Nettoyage complet

```bash
docker compose down --volumes --rmi all
```

---

## Liens utiles

- [Squash TM Community](https://www.squashtest.org/)
- [Documentation Orchestrator DEVOPS](https://tm-en.doc.squashtest.com/automation/orchestrator.html)
- [Jenkins Docker Docs](https://www.jenkins.io/doc/book/installing/docker/)
- [Mailhog (test emails)](https://github.com/mailhog/MailHog)
