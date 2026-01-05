# 🚀 Guide Complet - Projet CI/CD pour BFB Management

**Date limite : Présentation le 9 janvier 2026**  
**Objectif : Pipeline CI/CD avec déploiement automatique et rollback**

---

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Plan détaillé du projet](#plan-détaillé-du-projet)
3. [Configuration de l'environnement](#configuration-de-lenvironnement)
4. [Développement de la plateforme CI/CD](#développement-de-la-plateforme-cicd)
5. [Setup de la VM](#setup-de-la-vm)
6. [Tests et validation](#tests-et-validation)
7. [Préparation de la présentation](#préparation-de-la-présentation)
8. [Commandes utiles](#commandes-utiles)

---

## 🎯 Vue d'ensemble

### Objectifs du projet

Ce projet consiste à créer une **plateforme CI/CD complète** qui automatise :
- ✅ La récupération du code depuis GitHub
- ✅ La compilation et les tests (Maven)
- ✅ L'analyse de code (SonarQube)
- ✅ La création d'images Docker
- ✅ Le déploiement sur une VM via SSH
- ✅ Le rollback automatique en cas d'erreur

### Architecture globale

```
┌─────────────────┐
│   GitHub Repo   │
│  (Application)  │
└────────┬────────┘
         │ Webhook
         ↓
┌─────────────────────────────────┐
│   Plateforme CI/CD              │
│  ┌──────────┐   ┌───────────┐  │
│  │ Frontend │←──│  Backend  │  │
│  │  React   │   │  Node.js  │  │
│  └──────────┘   └─────┬─────┘  │
│                       │         │
│                 ┌─────┴─────┐   │
│                 │  Redis    │   │
│                 │  Queue    │   │
│                 └─────┬─────┘   │
│                       │         │
│                 ┌─────┴─────┐   │
│                 │ Pipeline  │   │
│                 │ Executor  │   │
│                 └─────┬─────┘   │
└───────────────────────┼─────────┘
                        │ SSH
                        ↓
              ┌─────────────────┐
              │   VM Ubuntu     │
              │   + Docker      │
              │   + App         │
              └─────────────────┘
```

---

## 📝 Plan détaillé du projet

### ✅ PHASE 1 : Préparation de l'application (Jours 1-2)

#### 🔹 1.1 Dockerisation de l'application

**Objectif :** Rendre l'application déployable via Docker

**Étape 1 : Créer le Dockerfile**

```dockerfile
# Dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Étape 2 : Créer docker-compose.yml**

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: bfb_db
      POSTGRES_USER: bfb_user
      POSTGRES_PASSWORD: bfb_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - bfb-network

  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/bfb_db
      SPRING_DATASOURCE_USERNAME: bfb_user
      SPRING_DATASOURCE_PASSWORD: bfb_pass
    depends_on:
      - db
    networks:
      - bfb-network

volumes:
  postgres_data:

networks:
  bfb-network:
```

**Étape 3 : Tester localement**

```bash
docker-compose up --build
curl http://localhost:8080/api/v1/clients
```

**✅ Validation :** L'application démarre et répond aux requêtes HTTP

---

#### 🔹 1.2 Tests Unitaires

**Objectif :** S'assurer que les tests passent avant déploiement

**Étape 1 : Vérifier les tests existants**

```bash
mvn clean test
```

**Étape 2 : Ajouter des tests si nécessaire**

Les tests sont déjà présents dans `src/test/`. Si besoin, ajouter des tests pour :
- `ClientService`
- `VehicleService`
- `ContractService`

**Étape 3 : Configurer le rapport de couverture**

Ajouter dans `pom.xml` :

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**✅ Validation :** `mvn test` passe avec succès

---

#### 🔹 1.3 Configuration SonarQube

**Objectif :** Analyser la qualité du code

**Étape 1 : Lancer SonarQube en local**

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
```

Accéder à http://localhost:9000 (login: admin/admin)

**Étape 2 : Créer un projet et générer un token**

1. Créer un projet "bfb-management"
2. Générer un token d'authentification
3. Sauvegarder le token

**Étape 3 : Créer sonar-project.properties**

```properties
sonar.projectKey=bfb-management
sonar.projectName=BFB Management System
sonar.projectVersion=1.0
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
sonar.java.test.binaries=target/test-classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

**Étape 4 : Lancer l'analyse**

```bash
mvn clean verify sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN
```

**✅ Validation :** Le rapport apparaît dans SonarQube

---

### ✅ PHASE 2 : Développement de la plateforme CI/CD (Jours 3-5)

#### 🔹 2.1 Setup du projet CI/CD

**Architecture technique choisie :**

```
Backend: Node.js + Express
Frontend: React + Vite
Base de données: PostgreSQL
Queue: Redis (Bull)
WebSockets: Socket.io
Authentification: Passport.js (OAuth2)
```

**Étape 1 : Créer la structure**

```bash
mkdir cicd-platform
cd cicd-platform

# Backend
mkdir backend
cd backend
npm init -y
npm install express pg redis bull socket.io passport passport-github2 jsonwebtoken bcrypt ssh2 dotenv cors

# Frontend
cd ..
npm create vite@latest frontend -- --template react
cd frontend
npm install axios socket.io-client react-router-dom
```

---

#### 🔹 2.2 Backend - API REST

**Structure du backend :**

```
backend/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   ├── redis.js
│   │   └── passport.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Pipeline.js
│   │   └── PipelineLog.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── pipelines.js
│   │   └── webhooks.js
│   ├── services/
│   │   ├── pipelineExecutor.js
│   │   ├── sshService.js
│   │   └── githubService.js
│   ├── middleware/
│   │   ├── auth.js
│   │   └── roles.js
│   └── server.js
├── .env
└── package.json
```

**Schéma de base de données :**

```sql
-- users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    github_id VARCHAR(255) UNIQUE,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    role VARCHAR(50) DEFAULT 'viewer',
    created_at TIMESTAMP DEFAULT NOW()
);

-- pipelines table
CREATE TABLE pipelines (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    status VARCHAR(50) DEFAULT 'pending',
    commit_hash VARCHAR(255),
    branch VARCHAR(255),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- pipeline_logs table
CREATE TABLE pipeline_logs (
    id SERIAL PRIMARY KEY,
    pipeline_id INTEGER REFERENCES pipelines(id),
    step_name VARCHAR(255),
    status VARCHAR(50),
    output TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Exemple de code - Pipeline Executor (backend/src/services/pipelineExecutor.js) :**

```javascript
const { exec } = require('child_process');
const util = require('util');
const execPromise = util.promisify(exec);
const sshService = require('./sshService');
const db = require('../config/database');
const io = require('../server').io;

const PIPELINE_STEPS = [
  {
    name: 'Clone Repository',
    execute: async (repoUrl, workDir) => {
      const { stdout, stderr } = await execPromise(`git clone ${repoUrl} ${workDir}`);
      return stdout || stderr;
    }
  },
  {
    name: 'Run Tests',
    execute: async (workDir) => {
      const { stdout, stderr } = await execPromise(`cd ${workDir} && mvn clean test`);
      return stdout || stderr;
    }
  },
  {
    name: 'Build Maven Package',
    execute: async (workDir) => {
      const { stdout, stderr } = await execPromise(`cd ${workDir} && mvn package -DskipTests`);
      return stdout || stderr;
    }
  },
  {
    name: 'SonarQube Analysis',
    execute: async (workDir) => {
      const { stdout, stderr } = await execPromise(
        `cd ${workDir} && mvn sonar:sonar -Dsonar.host.url=${process.env.SONAR_URL} -Dsonar.login=${process.env.SONAR_TOKEN}`
      );
      return stdout || stderr;
    }
  },
  {
    name: 'Build Docker Image',
    execute: async (workDir, version) => {
      const { stdout, stderr } = await execPromise(
        `cd ${workDir} && docker build -t bfb-management:${version} .`
      );
      return stdout || stderr;
    }
  },
  {
    name: 'Deploy to VM',
    execute: async (version) => {
      return await sshService.deployToVM(version);
    }
  },
  {
    name: 'Health Check',
    execute: async () => {
      const { stdout } = await execPromise(`curl http://${process.env.VM_IP}:8080/actuator/health`);
      return stdout;
    }
  }
];

async function executePipeline(pipelineId, repoUrl, commitHash) {
  const workDir = `/tmp/pipeline-${pipelineId}`;
  const version = `v${Date.now()}`;
  
  try {
    await db.query('UPDATE pipelines SET status = $1, started_at = NOW() WHERE id = $2', ['running', pipelineId]);
    
    for (const step of PIPELINE_STEPS) {
      io.to(`pipeline-${pipelineId}`).emit('step_started', { step: step.name });
      
      try {
        const output = await step.execute(workDir, version);
        
        await db.query(
          'INSERT INTO pipeline_logs (pipeline_id, step_name, status, output) VALUES ($1, $2, $3, $4)',
          [pipelineId, step.name, 'success', output]
        );
        
        io.to(`pipeline-${pipelineId}`).emit('step_completed', {
          step: step.name,
          status: 'success',
          output
        });
      } catch (error) {
        await db.query(
          'INSERT INTO pipeline_logs (pipeline_id, step_name, status, output) VALUES ($1, $2, $3, $4)',
          [pipelineId, step.name, 'failed', error.message]
        );
        
        io.to(`pipeline-${pipelineId}`).emit('step_failed', {
          step: step.name,
          error: error.message
        });
        
        // Rollback
        await rollbackPipeline(pipelineId);
        throw error;
      }
    }
    
    await db.query('UPDATE pipelines SET status = $1, completed_at = NOW() WHERE id = $2', ['success', pipelineId]);
    io.to(`pipeline-${pipelineId}`).emit('pipeline_completed', { status: 'success' });
    
  } catch (error) {
    await db.query('UPDATE pipelines SET status = $1, completed_at = NOW() WHERE id = $2', ['failed', pipelineId]);
    io.to(`pipeline-${pipelineId}`).emit('pipeline_failed', { error: error.message });
  }
}

async function rollbackPipeline(pipelineId) {
  // Récupérer la dernière version réussie
  const result = await db.query(
    'SELECT commit_hash FROM pipelines WHERE status = $1 ORDER BY completed_at DESC LIMIT 1',
    ['success']
  );
  
  if (result.rows.length > 0) {
    const previousVersion = result.rows[0].commit_hash;
    await sshService.deployToVM(`v${previousVersion}`);
    io.to(`pipeline-${pipelineId}`).emit('rollback_completed', { version: previousVersion });
  }
}

module.exports = { executePipeline, rollbackPipeline };
```

---

#### 🔹 2.3 Frontend - Interface React

**Pages principales :**

1. **Login Page** (`/login`)
   - Bouton "Se connecter avec GitHub"
   - Redirection OAuth2

2. **Dashboard** (`/`)
   - Liste des pipelines récents
   - Statistiques (succès/échecs)
   - Bouton "Nouveau déploiement"

3. **Pipeline Detail** (`/pipeline/:id`)
   - Étapes du pipeline en temps réel
   - Logs scrollables
   - Bouton rollback

**Exemple de composant - Pipeline Viewer :**

```jsx
// src/components/PipelineViewer.jsx
import { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import io from 'socket.io-client';

export default function PipelineViewer() {
  const { id } = useParams();
  const [steps, setSteps] = useState([]);
  const [logs, setLogs] = useState([]);
  const [status, setStatus] = useState('pending');

  useEffect(() => {
    const socket = io('http://localhost:3001');
    
    socket.emit('subscribe', `pipeline-${id}`);
    
    socket.on('step_started', (data) => {
      setSteps(prev => [...prev, { name: data.step, status: 'running' }]);
    });
    
    socket.on('step_completed', (data) => {
      setSteps(prev => prev.map(s => 
        s.name === data.step ? { ...s, status: 'success' } : s
      ));
      setLogs(prev => [...prev, data.output]);
    });
    
    socket.on('step_failed', (data) => {
      setSteps(prev => prev.map(s => 
        s.name === data.step ? { ...s, status: 'failed' } : s
      ));
      setLogs(prev => [...prev, `ERROR: ${data.error}`]);
    });
    
    socket.on('pipeline_completed', () => {
      setStatus('success');
    });
    
    socket.on('pipeline_failed', () => {
      setStatus('failed');
    });
    
    return () => socket.disconnect();
  }, [id]);

  return (
    <div className="pipeline-viewer">
      <h1>Pipeline #{id}</h1>
      <div className={`status ${status}`}>{status.toUpperCase()}</div>
      
      <div className="steps">
        {steps.map((step, i) => (
          <div key={i} className={`step ${step.status}`}>
            <span className="step-icon">
              {step.status === 'running' && '⏳'}
              {step.status === 'success' && '✅'}
              {step.status === 'failed' && '❌'}
            </span>
            <span>{step.name}</span>
          </div>
        ))}
      </div>
      
      <div className="logs">
        <h3>Logs</h3>
        <pre>
          {logs.join('\n')}
        </pre>
      </div>
    </div>
  );
}
```

---

### ✅ PHASE 3 : Configuration de la VM (Jours 5-6)

#### 🔹 3.1 Installation de la VM

**Étape 1 : Télécharger Ubuntu Server**

- URL : https://ubuntu.com/download/server
- Version : 22.04 LTS (ISO)

**Étape 2 : Créer la VM dans VirtualBox**

```
Nom : BFB-Production
Type : Linux
Version : Ubuntu (64-bit)
RAM : 4096 MB
Disque : 20 GB (dynamique)
Réseau : Mode Bridge (ou NAT avec redirection de ports)
```

**Étape 3 : Installer Ubuntu**

1. Démarrer la VM avec l'ISO
2. Suivre l'installation minimale
3. Créer un utilisateur `deployer`
4. Installer OpenSSH Server (cocher lors de l'installation)

---

#### 🔹 3.2 Configuration de la VM

**Se connecter à la VM :**

```bash
ssh deployer@<VM_IP>
```

**Installation des outils :**

```bash
# Mise à jour
sudo apt update && sudo apt upgrade -y

# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker deployer

# Installer Docker Compose
sudo apt install docker-compose -y

# Vérifier
docker --version
docker-compose --version
```

**Configuration SSH (sur votre machine locale) :**

```bash
# Générer une clé SSH
ssh-keygen -t rsa -b 4096 -f ~/.ssh/vm_deployer -N ""

# Copier la clé sur la VM
ssh-copy-id -i ~/.ssh/vm_deployer.pub deployer@<VM_IP>

# Tester
ssh -i ~/.ssh/vm_deployer deployer@<VM_IP>
```

**Créer la structure sur la VM :**

```bash
mkdir -p ~/apps/bfb-management
mkdir -p ~/backups
```

**Créer le docker-compose.yml sur la VM :**

```bash
nano ~/apps/bfb-management/docker-compose.yml
```

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: bfb_db
      POSTGRES_USER: bfb_user
      POSTGRES_PASSWORD: bfb_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  app:
    image: bfb-management:latest
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/bfb_db
      SPRING_DATASOURCE_USERNAME: bfb_user
      SPRING_DATASOURCE_PASSWORD: bfb_pass
    depends_on:
      - db

volumes:
  postgres_data:
```

---

#### 🔹 3.3 Service SSH pour déploiement

**Créer le module SSH (backend/src/services/sshService.js) :**

```javascript
const { Client } = require('ssh2');

async function deployToVM(version) {
  const conn = new Client();
  
  return new Promise((resolve, reject) => {
    conn.on('ready', () => {
      const commands = [
        `cd ~/apps/bfb-management`,
        `docker-compose down`,
        `docker pull bfb-management:${version}`,
        `docker tag bfb-management:${version} bfb-management:latest`,
        `docker-compose up -d`,
        `sleep 5`,
        `curl -f http://localhost:8080/actuator/health || exit 1`
      ];
      
      conn.exec(commands.join(' && '), (err, stream) => {
        if (err) return reject(err);
        
        let output = '';
        stream.on('data', (data) => {
          output += data.toString();
        });
        
        stream.on('close', (code) => {
          conn.end();
          if (code === 0) {
            resolve(output);
          } else {
            reject(new Error(`Deployment failed with code ${code}`));
          }
        });
      });
    }).connect({
      host: process.env.VM_IP,
      port: 22,
      username: 'deployer',
      privateKey: require('fs').readFileSync(process.env.SSH_KEY_PATH)
    });
  });
}

module.exports = { deployToVM };
```

---

### ✅ PHASE 4 : Tests et Validation (Jours 7-8)

#### 🔹 4.1 Tests du pipeline complet

**Test 1 : Déploiement manuel**

```bash
# Depuis l'interface CI/CD
1. Se connecter
2. Cliquer sur "Nouveau déploiement"
3. Sélectionner la branche main
4. Observer les étapes en temps réel
5. Vérifier sur http://<VM_IP>:8080/api/v1/clients
```

**Test 2 : Déploiement automatique (webhook)**

```bash
# Configurer le webhook GitHub
URL: http://<YOUR_IP>:3001/api/webhooks/github
Secret: <VOTRE_SECRET>
Events: push

# Faire un commit
git commit -m "Test webhook"
git push origin main

# Observer le pipeline se déclencher automatiquement
```

**Test 3 : Rollback**

```bash
# Introduire une erreur volontaire
echo "INVALID_CONFIG=true" >> application.yml
git commit -m "Bug volontaire"
git push

# Observer l'échec du health check
# Observer le rollback automatique vers la version précédente
```

---

#### 🔹 4.2 Scénario de démonstration

**Pour la présentation :**

```
1. État initial : V1 déployée (sans DELETE)
   - Montrer que DELETE /clients/{id} retourne 404

2. Déployer V2 (avec DELETE)
   - Décommenter les routes
   - Push sur GitHub
   - Observer le pipeline en temps réel
   - Tester DELETE /clients/{id} → fonctionne

3. Déployer V3 (avec bug)
   - Introduire une erreur de config
   - Push sur GitHub
   - Observer l'échec
   - Observer le rollback automatique vers V2
   - Vérifier que V2 est toujours active
```

---

### ✅ PHASE 5 : Préparation de la présentation (Jours 9-10)

#### 🔹 5.1 Structure de la présentation (15 min)

**Slide 1-2 : Introduction (2 min)**
- Contexte du projet
- Objectifs (automatisation, qualité, rollback)
- Architecture globale (schéma)

**Slide 3-5 : Architecture technique (3 min)**
- Stack choisie (Node.js, React, PostgreSQL, Redis)
- Pipeline en 8 étapes
- Gestion du rollback

**Démonstration en direct (8 min)**
1. Connexion à la plateforme (OAuth2)
2. Déclenchement d'un déploiement
3. Suivi temps réel des étapes
4. Test sur la VM
5. Démo du rollback

**Slide 6-7 : Points techniques (2 min)**
- Sécurité (SSH, tokens, secrets)
- Difficultés rencontrées
- Améliorations possibles

---

#### 🔹 5.2 Checklist finale

**24h avant la présentation :**

- [ ] VM allumée et accessible
- [ ] Plateforme CI/CD lancée
- [ ] Base de données initialisée
- [ ] GitHub repo synchronisé
- [ ] Webhooks configurés et testés
- [ ] Compte OAuth2 fonctionnel
- [ ] Tests de bout en bout réussis (3x minimum)
- [ ] Support de présentation finalisé
- [ ] Vidéo de backup enregistrée (en cas de problème réseau)
- [ ] Script de démo préparé

**Le jour J :**

- [ ] Arriver 15 min avant
- [ ] Vérifier la connexion réseau
- [ ] Tester le projecteur
- [ ] Lancer tous les services
- [ ] Préparer un compte de test
- [ ] Avoir un plan B (vidéo)

---

## 🛠️ Commandes utiles

### Maven
```bash
# Compilation
mvn clean package

# Tests
mvn test

# SonarQube
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=TOKEN
```

### Docker
```bash
# Build
docker build -t bfb-management:v1 .

# Run
docker-compose up -d

# Logs
docker logs -f <container_name>

# Save/Load
docker save bfb-management:v1 > backup.tar
docker load < backup.tar

# Cleanup
docker system prune -a
```

### Git
```bash
# Init
git init
git add .
git commit -m "V1: Initial version"

# Remote
git remote add origin https://github.com/USER/REPO.git
git push -u origin main

# Tags
git tag v1.0
git push --tags
```

### SSH/VM
```bash
# Connexion
ssh -i ~/.ssh/vm_deployer deployer@VM_IP

# Copier un fichier
scp -i ~/.ssh/vm_deployer file.tar deployer@VM_IP:~/backups/

# Exécuter une commande
ssh -i ~/.ssh/vm_deployer deployer@VM_IP "docker ps"
```

---

## 📊 Critères d'évaluation

### Fonctionnalités (40%)
- ✅ Pipeline complet avec 8 étapes
- ✅ OAuth2 fonctionnel
- ✅ Gestion des rôles (admin/deployer/viewer)
- ✅ Rollback automatique
- ✅ Logs en temps réel

### Technique (30%)
- ✅ Code propre et structuré
- ✅ Architecture cohérente
- ✅ Sécurité (secrets, SSH, HTTPS)
- ✅ Tests unitaires
- ✅ Documentation

### Présentation (30%)
- ✅ Clarté des explications
- ✅ Démo fluide
- ✅ Maîtrise du sujet
- ✅ Réponses aux questions

---

## 📚 Ressources complémentaires

### Documentation officielle
- [Docker Documentation](https://docs.docker.com/)
- [SonarQube Docs](https://docs.sonarqube.org/)
- [GitHub OAuth Apps](https://docs.github.com/en/developers/apps)
- [Node.js SSH2](https://github.com/mscdex/ssh2)
- [Socket.io](https://socket.io/docs/)

### Tutoriels
- [CI/CD avec GitHub Actions](https://docs.github.com/en/actions)
- [Déploiement Docker via SSH](https://www.digitalocean.com/community/tutorials)
- [React + WebSockets](https://socket.io/how-to/use-with-react)

---

## 🎓 Conseils finaux

**⚠️ Prioriser la simplicité**
- Ne cherchez pas à faire le projet parfait
- Concentrez-vous sur les fonctionnalités demandées
- Préférez une solution simple qui fonctionne à une solution complexe qui bug

**⚠️ Tester, tester, tester**
- Testez chaque étape individuellement
- Testez le pipeline complet au moins 3 fois
- Préparez un plan B pour la démo

**⚠️ Documenter au fur et à mesure**
- Prenez des notes pendant le développement
- Capturez des screenshots
- Gardez une trace des problèmes résolus

**⚠️ Répartir le travail efficacement**
- Personne 1 : Application + Docker + Tests
- Personne 2 : Backend CI/CD + Pipeline
- Personne 3 : Frontend + UI/UX
- Personne 4 : VM + SSH + Intégration

---

## 👥 Organisation recommandée

### Planning jour par jour

**Jour 1 (5 janvier)**
- Réunion de lancement (1h)
- Répartition des tâches
- Setup des environnements
- Début Docker + Tests

**Jour 2 (6 janvier)**
- Finalisation Docker
- Configuration SonarQube
- Début backend CI/CD

**Jour 3 (7 janvier)**
- Backend : API + Pipeline executor
- Frontend : Structure de base
- VM : Installation

**Jour 4 (8 janvier)**
- Backend : WebSockets + OAuth2
- Frontend : Interface complète
- VM : Configuration SSH

**Jour 5 (9 janvier)**
- Intégration complète
- Tests end-to-end
- Fix des bugs

**Jour 6 (10 janvier)**
- Tests du rollback
- Optimisations
- Documentation

**Jour 7 (11 janvier)**
- Tests finaux
- Préparation de la présentation
- Répétition

**Jour 8 (12 janvier)**
- Derniers ajustements
- Répétition finale
- Backup de la démo

**Jour 9 (13 janvier)**
- **PRÉSENTATION** 🎉

---

## ✅ Checklist complète

### Phase 1 : Application
- [ ] Dockerfile créé et testé
- [ ] docker-compose.yml fonctionnel
- [ ] Tests unitaires passent
- [ ] SonarQube configuré
- [ ] Analyse de code effectuée

### Phase 2 : Plateforme CI/CD
- [ ] Backend : Structure créée
- [ ] Backend : Base de données configurée
- [ ] Backend : OAuth2 fonctionnel
- [ ] Backend : Pipeline executor implémenté
- [ ] Backend : WebSockets actifs
- [ ] Backend : Webhook GitHub
- [ ] Frontend : Pages créées
- [ ] Frontend : Connexion temps réel
- [ ] Frontend : Interface intuitive

### Phase 3 : Infrastructure
- [ ] VM créée et installée
- [ ] Docker installé sur VM
- [ ] SSH configuré
- [ ] Connexion sans mot de passe
- [ ] Structure de dossiers créée
- [ ] docker-compose.yml sur VM

### Phase 4 : Intégration
- [ ] Déploiement manuel testé
- [ ] Déploiement automatique testé
- [ ] Rollback testé
- [ ] Logs en temps réel testés
- [ ] Gestion des rôles testée
- [ ] Pipeline complet testé 3x

### Phase 5 : Présentation
- [ ] Support créé
- [ ] Démo préparée
- [ ] Vidéo de backup
- [ ] Répétition effectuée
- [ ] Questions anticipées
- [ ] Plan B préparé

---

**Bon courage pour votre projet ! 🚀**

*Ce document doit vous servir de guide tout au long du projet et lors de votre présentation.*
