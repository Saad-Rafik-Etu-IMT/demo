# 🚀 CI/CD Platform - BFB Management

**Deadline:** 9 janvier 2026  
**Objectif:** Pipeline automatisé avec déploiement et rollback

---

## 🎯 Objectifs

Pipeline CI/CD qui automatise :
- Récupération du code (GitHub)
- Compilation et tests (Maven)
- Analyse de qualité (SonarQube)
- Création d'images Docker
- Déploiement VM via SSH
- Rollback automatique

## 📐 Architecture

```
GitHub → CI/CD Platform (Node.js + React + Redis) → VM Ubuntu (Docker)
```

**Stack Technique:**
- App: Java Spring Boot + PostgreSQL
- CI/CD Backend: Node.js + Express + Redis + Socket.io
- CI/CD Frontend: React + Vite
- Infrastructure: Docker + VM Ubuntu

**Pipeline (7 étapes):**
1. Clone Repository
2. Run Tests (Maven)
3. Build Package
4. SonarQube Analysis
5. Build Docker Image
6. Deploy to VM (SSH)
7. Health Check + Rollback si échec

---

## 📝 Phase 1: Application (Jour 1)

### 1.1 Dockerisation

**Dockerfile:**
```dockerfile
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

**docker-compose.yml:**
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

**Test:** `docker-compose up --build`

### 1.2 Tests et Qualité

**Ajouter JaCoCo dans pom.xml:**
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution><goals><goal>prepare-agent</goal></goals></execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
</plugin>
```

**SonarQube:**
```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
```

**sonar-project.properties:**
```properties
sonar.projectKey=bfb-management
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

---

## 📝 Phase 2: CI/CD Platform (Jours 2-4)

### 2.1 Backend Structure

**Setup:**
```bash
mkdir cicd-platform && cd cicd-platform
mkdir backend && cd backend
npm init -y
npm install express pg redis bull socket.io ssh2 dotenv cors
```

**Architecture:**
```
backend/
├── src/
│   ├── config/        # database.js, redis.js
│   ├── routes/        # pipelines.js, webhooks.js
│   ├── services/      # pipelineExecutor.js, sshService.js
│   └── server.js
└── .env
```

**Base de données (PostgreSQL):**
```sql
CREATE TABLE pipelines (
    id SERIAL PRIMARY KEY,
    status VARCHAR(50) DEFAULT 'pending',
    commit_hash VARCHAR(255),
    branch VARCHAR(255),
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);

CREATE TABLE pipeline_logs (
    id SERIAL PRIMARY KEY,
    pipeline_id INTEGER REFERENCES pipelines(id),
    step_name VARCHAR(255),
    status VARCHAR(50),
    output TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 2.2 Pipeline Executor (backend/src/services/pipelineExecutor.js)

```javascript
const { exec } = require('child_process');
const util = require('util');
const execPromise = util.promisify(exec);

const STEPS = [
  { name: 'Clone', cmd: (repoUrl, dir) => `git clone ${repoUrl} ${dir}` },
  { name: 'Test', cmd: (dir) => `cd ${dir} && mvn clean test` },
  { name: 'Build', cmd: (dir) => `cd ${dir} && mvn package -DskipTests` },
  { name: 'Docker', cmd: (dir, v) => `cd ${dir} && docker build -t bfb:${v} .` },
  { name: 'Deploy', cmd: async (v) => await deployToVM(v) },
  { name: 'Health', cmd: () => `curl http://${process.env.VM_IP}:8080/actuator/health` }
];

async function executePipeline(pipelineId, repoUrl) {
  const dir = `/tmp/pipeline-${pipelineId}`;
  const version = `v${Date.now()}`;
  
  try {
    for (const step of STEPS) {
      const output = await execPromise(step.cmd(repoUrl, dir, version));
      // Log to DB + emit via Socket.io
    }
  } catch (error) {
    await rollback(pipelineId);
  }
}
```

### 2.3 SSH Service (backend/src/services/sshService.js)

```javascript
const { Client } = require('ssh2');

async function deployToVM(version) {
  const conn = new Client();
  return new Promise((resolve, reject) => {
    conn.on('ready', () => {
      const cmd = `cd ~/apps/bfb && docker-compose down && ` +
                  `docker tag bfb:${version} bfb:latest && ` +
                  `docker-compose up -d`;
      conn.exec(cmd, (err, stream) => {
        stream.on('close', (code) => {
          conn.end();
          code === 0 ? resolve() : reject();
        });
      });
    }).connect({
      host: process.env.VM_IP,
      username: 'deployer',
      privateKey: require('fs').readFileSync(process.env.SSH_KEY_PATH)
    });
  });
}
```

### 2.4 Frontend (React)

```bash
npm create vite@latest frontend -- --template react
cd frontend
npm install axios socket.io-client react-router-dom
```

**Pages:**
- `/` - Dashboard (liste pipelines)
- `/pipeline/:id` - Détails temps réel (WebSocket)

---

## 📝 Phase 3: VM Setup (Jour 3-4)

### 3.1 Installation VM

1. Télécharger Ubuntu Server 22.04 LTS
2. VirtualBox: 4GB RAM, 20GB disque, mode Bridge
3. Créer utilisateur `deployer` + installer OpenSSH

### 3.2 Configuration VM

```bash
# Sur VM
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker deployer
sudo apt install docker-compose -y

mkdir -p ~/apps/bfb-management ~/backups
```

### 3.3 SSH Setup

```bash
# Sur machine locale
ssh-keygen -t rsa -b 4096 -f ~/.ssh/vm_deployer -N ""
ssh-copy-id -i ~/.ssh/vm_deployer.pub deployer@VM_IP
```

**docker-compose.yml sur VM:**
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
    image: bfb:latest
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/bfb_db
    depends_on:
      - db

volumes:
  postgres_data:
```

---

## 📝 Phase 4: Tests (Jour 5)

### Tests Critiques

1. **Déploiement manuel** via interface
2. **Webhook GitHub** (push → déploiement auto)
3. **Rollback** (introduire bug → vérifier retour V précédente)

### Scénario Démo

1. V1 déployée → Tester API
2. Push V2 → Observer pipeline temps réel
3. Push V3 (avec bug) → Observer rollback automatique

---

## 🛠️ Commandes Essentielles

```bash
# Maven
mvn clean test
mvn package -DskipTests
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=TOKEN

# Docker
docker build -t bfb:v1 .
docker-compose up -d
docker logs -f <container>

# Git
git add . && git commit -m "message" && git push

# SSH/VM
ssh -i ~/.ssh/vm_deployer deployer@VM_IP
scp -i ~/.ssh/vm_deployer file.tar deployer@VM_IP:~/backups/
```

---

## ✅ Checklist Finale

### Application
- [ ] Dockerfile testé
- [ ] docker-compose fonctionnel
- [ ] Tests passent (`mvn test`)
- [ ] SonarQube configuré

### CI/CD Platform
- [ ] Backend: API + Pipeline executor
- [ ] Frontend: Dashboard + Pipeline viewer
- [ ] WebSockets temps réel
- [ ] Webhook GitHub

### Infrastructure
- [ ] VM Ubuntu opérationnelle
- [ ] Docker installé
- [ ] SSH sans mot de passe
- [ ] docker-compose.yml sur VM

### Validation
- [ ] Pipeline complet testé (3x)
- [ ] Rollback fonctionnel
- [ ] Démo préparée
- [ ] Support présentation prêt

---

## 🎓 Conseils Critiques

**Simplifier si nécessaire:**
- OAuth2 → Login basique
- SonarQube → Optionnel
- Redis → File system si problème

**Priorités:**
1. Pipeline fonctionnel bout en bout
2. Rollback automatique
3. Interface temps réel
4. Qualité du code (si temps)

**Backup:**
- Enregistrer vidéo de la démo
- Préparer Plan B (démo locale)
- Tester 24h avant présentation

---

**Deadline: 9 janvier 2026 🚀**
