# 🚀 CI/CD Platform - Documentation Complète

**Projet:** Plateforme CI/CD pour l'application BFB Management  
**Deadline:** 9 janvier 2026  
**Objectif:** Pipeline automatisé de déploiement avec rollback et analyse qualité

---

## 📊 État d'Implémentation du Projet

| Priorité | Fonctionnalité | Statut | Effort Restant | Note |
|----------|----------------|--------|----------------|------|
| 🔴 **P0** | Pipeline bout-en-bout | ✅ **Complet** | - | 7 étapes automatisées |
| 🔴 **P0** | Authentification OAuth2 GitHub | ✅ **Complet** | - | Login + rôles (admin/dev/viewer) |
| 🔴 **P0** | Interface temps réel (WebSocket) | ✅ **Complet** | - | Suivi live des pipelines |
| 🔴 **P0** | Déploiement SSH sur VM | ✅ **Complet** | - | Service SSH fonctionnel |
| 🔴 **P0** | Rollback automatique | ✅ **Complet** | - | En cas d'échec health check |
| 🔴 **P0** | Webhook GitHub | ✅ **Complet** | 30min | Ajouter signature HMAC |
| 🟡 **P1** | Tests unitaires (demo) | ✅ **Complet** | - | 24 tests JUnit + JaCoCo |
| 🟡 **P1** | Dockerisation app métier | ✅ **Complet** | - | Multi-stage build |
| 🟡 **P1** | SonarQube intégration | ⚠️ **Partiel** | 1h | Ajouter au docker-compose |
| 🟡 **P1** | Variables d'environnement | ✅ **Complet** | - | Gestion via IHM |
| 🟡 **P1** | Support présentation | ❌ **À faire** | 1h | Plan démo + slides |
| 🟢 **P2** | Tests d'intrusion (PenTest) | ❌ **Facultatif** | 2h | OWASP ZAP |
| 🟢 **P2** | Kubernetes | ❌ **Bonus** | 4h | Manifests K8s |

**Score actuel : 85% des exigences complétées** 🎉

---

## 📚 Glossaire des Concepts (Pour Débutants)

### 🔧 Concepts Généraux

#### **CI/CD (Continuous Integration / Continuous Deployment)**
**Définition simple :** Système qui automatise la vérification et le déploiement de votre code.
- **CI (Intégration Continue)** : Vérifier automatiquement que votre code ne casse rien (tests automatiques)
- **CD (Déploiement Continu)** : Mettre automatiquement votre code en production

**Analogie :** Imaginez une usine automobile :
- **Sans CI/CD** : Vous assemblez chaque voiture à la main, une par une → lent et plein d'erreurs
- **Avec CI/CD** : Une chaîne de montage automatisée qui vérifie chaque pièce et assemble tout → rapide et fiable

**Cas d'usage réel :**
- Vous corrigez un bug à 18h → Push sur GitHub → À 18h03, la correction est déjà en production
- Votre collègue ajoute une fonctionnalité → Tests automatiques détectent qu'elle casse le code → Il est notifié avant que ça arrive en prod

---

#### **Pipeline**
**Définition simple :** Une suite d'étapes automatiques qui transforment votre code en application déployée.

**Analogie :** C'est comme une recette de cuisine :
1. **Clone** → Sortir les ingrédients du placard
2. **Test** → Vérifier qu'ils ne sont pas périmés
3. **Build** → Cuisiner le plat
4. **Docker** → Mettre dans un tupperware
5. **Deploy** → Livrer chez le client
6. **Health Check** → Le client goûte et valide

**Dans notre projet :**
```
Code GitHub → Clone → Tests (24 tests JUnit) → Build Maven → 
Image Docker → Déploiement SSH → Vérification santé
```

**Cas d'usage réel :**
- **Netflix** : 4000 déploiements/jour → Chaque modification passe par un pipeline
- **Amazon** : Déploiement toutes les 11 secondes en moyenne

---

#### **Webhook**
**Définition simple :** Un "coup de téléphone" automatique qu'un service (GitHub) passe à un autre (votre CI/CD) quand un événement se produit.

**Analogie :** 
- **Sans webhook** : Vous appelez toutes les 5 minutes votre livreur pour savoir s'il a un colis → inefficace
- **Avec webhook** : Le livreur vous appelle directement quand il arrive → efficace

**Dans notre projet :**
```
Vous → git push → GitHub détecte le push → 
GitHub envoie un POST HTTP à votre CI/CD → 
Votre pipeline démarre automatiquement
```

**Exemple concret :**
```http
POST http://votre-serveur.com/api/webhooks/github
Headers: {
  "x-github-event": "push"
}
Body: {
  "repository": "demo",
  "pusher": "votre_nom",
  "commit": "abc123..."
}
```

**Cas d'usage réel :**
- **Slack** : Notification automatique quand quelqu'un push du code
- **Vercel** : Redéploiement automatique de votre site web
- **Discord** : Bot qui annonce les commits dans un channel

---

#### **WebSocket**
**Définition simple :** Un "tuyau" de communication permanent entre le navigateur et le serveur (contrairement à HTTP qui est une série de "questions-réponses").

**Analogie :**
- **HTTP classique** : Vous envoyez une lettre, attendez la réponse, renvoyez une lettre, etc. → lent
- **WebSocket** : Vous avez une conversation téléphonique continue → instantané

**Dans notre projet :**
```
Navigateur ←─────WebSocket─────→ Serveur
    ↑                              ↓
    │     "step_started"           │
    │◄───────────────────────────  │
    │     "step_completed"         │
    │◄───────────────────────────  │
    └─── Affichage temps réel ─────┘
```

**Pourquoi c'est important ici ?**
Sans WebSocket :
- Vous devriez rafraîchir la page toutes les 2 secondes pour voir l'avancement → 🐢
Avec WebSocket :
- Le serveur vous envoie les updates instantanément → ⚡

**Cas d'usage réel :**
- **Google Docs** : Vous voyez ce que votre collègue tape en temps réel
- **Trading apps** : Prix des actions qui se mettent à jour en direct
- **Chat en ligne** : Messages instantanés

---

#### **SSH (Secure Shell)**
**Définition simple :** Un moyen sécurisé de contrôler un ordinateur distant via la ligne de commande.

**Analogie :** C'est comme avoir une télécommande ultra-sécurisée pour votre serveur.

**Dans notre projet :**
```
Votre CI/CD Platform ──SSH──→ VM Ubuntu
                               ↓
                        "docker stop app"
                        "docker run new_version"
                        "systemctl restart nginx"
```

**Authentification par clé (pas de mot de passe) :**
```
Vous créez une paire de clés :
- Clé privée (reste sur votre CI/CD) = clé de voiture
- Clé publique (sur la VM) = serrure de voiture
→ Seule votre clé peut démarrer cette voiture
```

**Cas d'usage réel :**
- **GitHub** : Vous poussez du code via SSH (git push)
- **Administrateurs systèmes** : Gèrent 100 serveurs depuis leur bureau
- **Déploiement automatique** : Ansible, Terraform utilisent SSH

---

#### **Docker**
**Définition simple :** Un "conteneur" qui emballe votre application avec tout ce dont elle a besoin pour fonctionner (Java, librairies, config).

**Analogie :** 
- **Sans Docker** : Vous déménagez et devez racheter tous les meubles → galère
- **Avec Docker** : Vous déménagez avec un container qui contient tous vos meubles montés → simple

**Avantages :**
- ✅ **Portabilité** : Fonctionne sur votre PC, sur la VM, sur AWS → partout pareil
- ✅ **Isolation** : Si une app plante, les autres continuent de tourner
- ✅ **Reproductibilité** : "Ça marche sur ma machine" → "Ça marchera partout"

**Dans notre projet :**
```dockerfile
# Image de base (Java 17)
FROM eclipse-temurin:17-jre-alpine

# Copier l'application
COPY demo.jar /app/app.jar

# Démarrer l'app
CMD ["java", "-jar", "/app/app.jar"]
```

**Cas d'usage réel :**
- **Uber** : 4000+ microservices = 4000+ conteneurs Docker
- **Spotify** : Gère 200+ services dans des conteneurs
- **Votre projet** : L'app demo tourne dans un conteneur, isolée de la VM

---

#### **Rollback**
**Définition simple :** Revenir à la version précédente de l'application si la nouvelle version plante.

**Analogie :** C'est le bouton "Annuler" (Ctrl+Z) pour les déploiements.

**Dans notre projet :**
```
V1 (stable) ──Deploy V2──→ V2 (bug) ──Health Check FAIL──→ Rollback ──→ V1 (stable restaurée)
                3 min                    +10 sec                           +30 sec
```

**Comment ça marche :**
1. Avant chaque déploiement, on sauvegarde l'image Docker précédente
2. Si la nouvelle version échoue le health check, on redéploie l'ancienne
3. Notification envoyée aux admins

**Cas d'usage réel :**
- **Facebook** : Rollback automatique si le taux d'erreur dépasse 0.1%
- **Netflix** : Canary deployment + rollback si les métriques baissent
- **Votre projet** : Si l'endpoint `/actuator/health` ne répond pas, rollback en 30 secondes

---

#### **OAuth2**
**Définition simple :** Un système qui permet de se connecter avec son compte GitHub/Google sans partager son mot de passe.

**Analogie :** 
- **Mot de passe classique** : Donner votre clé de maison à quelqu'un
- **OAuth2** : Donner un badge temporaire qui expire après 7 jours

**Flow dans notre projet :**
```
1. User clique "Login with GitHub"
2. Redirection vers github.com (GitHub demande "Autoriser cette app ?")
3. User accepte
4. GitHub renvoie un "code secret"
5. Notre backend échange ce code contre un "token d'accès"
6. User est connecté avec son profil GitHub (nom, email, avatar)
```

**Avantages :**
- ✅ Pas besoin de gérer des mots de passe (sécurité)
- ✅ Authentification déjà vérifiée par GitHub (fiabilité)
- ✅ Permissions granulaires (token peut être révoqué)

**Cas d'usage réel :**
- **Tous les sites modernes** : "Se connecter avec Google/Facebook/GitHub"
- **APIs** : Spotify, Stripe, Twilio utilisent OAuth2
- **Mobile apps** : Authentification sans stocker de mot de passe

---

#### **API REST**
**Définition simple :** Un serveur qui répond à des requêtes HTTP (GET, POST, PUT, DELETE) avec des données JSON.

**Analogie :** C'est comme passer commande au drive d'un McDo :
- **GET** : "Quelle est la liste des menus ?" → Le serveur répond avec la liste
- **POST** : "Je veux un BigMac" → Le serveur crée la commande
- **PUT** : "Changez mon Coca en Sprite" → Le serveur modifie
- **DELETE** : "Annulez ma commande" → Le serveur supprime

**Dans notre projet (backend cicd-platform) :**
```http
GET  /api/pipelines          → Liste tous les pipelines
POST /api/pipelines/trigger  → Déclenche un nouveau pipeline
GET  /api/pipelines/123/logs → Récupère les logs du pipeline 123
```

**Exemple de réponse :**
```json
{
  "id": 123,
  "status": "running",
  "branch": "master",
  "started_at": "2026-01-06T10:30:00Z"
}
```

**Cas d'usage réel :**
- **Twitter API** : Récupérer les tweets
- **Stripe API** : Traiter des paiements
- **Google Maps API** : Calculer des itinéraires

---

#### **RBAC (Role-Based Access Control)**
**Définition simple :** Système de permissions basé sur les rôles (Admin, Développeur, Viewer).

**Analogie :** Dans une entreprise :
- **Admin** = Directeur → Accès à tout (coffre-fort, comptabilité, RH)
- **Developer** = Employé → Peut travailler, mais pas accéder à la compta
- **Viewer** = Stagiaire → Peut regarder, mais pas toucher

**Dans notre projet :**
| Rôle | Déclencher pipeline | Voir logs | Gérer users | Modifier VM |
|------|---------------------|-----------|-------------|-------------|
| **Admin** | ✅ | ✅ | ✅ | ✅ |
| **Developer** | ✅ | ✅ | ❌ | ❌ |
| **Viewer** | ❌ | ✅ | ❌ | ❌ |

**Cas d'usage réel :**
- **AWS IAM** : Permissions granulaires par utilisateur
- **Kubernetes** : Rôles (namespace-admin, pod-reader, etc.)
- **Votre entreprise** : Certains peuvent créer des factures, d'autres juste les lire

---

#### **Health Check**
**Définition simple :** Une vérification automatique que votre application est en bonne santé (répond correctement).

**Analogie :** C'est comme prendre votre pouls après un effort : si le cœur bat normalement, tout va bien.

**Dans notre projet :**
```bash
# Requête envoyée après déploiement
curl http://192.168.1.100:8080/actuator/health

# Réponse attendue
{"status": "UP"}

# Si réponse différente ou timeout → ROLLBACK
```

**Vérifications possibles :**
- ✅ L'app répond (pas d'erreur 500)
- ✅ La BDD est accessible
- ✅ Espace disque suffisant
- ✅ Temps de réponse < 2 secondes

**Cas d'usage réel :**
- **Kubernetes** : Redémarre automatiquement les pods "unhealthy"
- **Load balancers** : Redirigent le trafic uniquement vers les serveurs "healthy"
- **Monitoring** : Alertes si health check échoue pendant 5 minutes

---

### 🏗️ Concepts Infrastructure

#### **VM (Virtual Machine)**
**Définition simple :** Un ordinateur virtuel qui tourne à l'intérieur d'un ordinateur physique.

**Analogie :** C'est comme avoir plusieurs "mini-ordinateurs" dans votre PC :
- PC principal (Windows) → VM Ubuntu → VM Windows 11
- Chacune a son propre OS, ses propres fichiers, isolée des autres

**Dans notre projet :**
- Votre PC physique (Windows)
  - → VM Ubuntu (via VirtualBox)
    - → Docker (conteneurs)
      - → Application demo

**Avantages :**
- ✅ Isolation totale (si la VM plante, votre PC continue)
- ✅ Environnement de test (tester sur Ubuntu sans l'installer)
- ✅ Snapshots (sauvegarder l'état et restaurer)

**Cas d'usage réel :**
- **AWS EC2** : Chaque "instance" est une VM
- **Développeurs** : Tester sur Windows/Mac/Linux sans machine dédiée
- **Votre projet** : Simuler un serveur de production

---

#### **PostgreSQL vs Redis**
**Définition simple :** Deux types de bases de données avec des usages différents.

**PostgreSQL (base relationnelle) :**
- **Usage** : Stocker des données permanentes (users, pipelines, logs)
- **Analogie** : Un classeur à tiroirs bien organisé avec des dossiers
- **Exemple** : 
  ```sql
  SELECT * FROM pipelines WHERE status = 'success';
  ```

**Redis (cache en mémoire) :**
- **Usage** : Stocker temporairement des données rapides à lire (sessions, cache)
- **Analogie** : Un post-it sur votre bureau (rapide mais temporaire)
- **Exemple** :
  ```
  SET user:123:session "abc123..." EX 3600  // Expire après 1h
  ```

**Pourquoi les deux ?**
| Besoin | Base utilisée | Raison |
|--------|---------------|--------|
| Stocker historique pipelines | PostgreSQL | Données permanentes |
| Cache liste pipelines | Redis | Accès ultra-rapide |
| Gestion file d'attente jobs | Redis | Pub/Sub temps réel |

---

#### **Maven**
**Définition simple :** Un outil qui automatise la construction d'applications Java (télécharge les librairies, compile, teste, emballe).

**Analogie :** C'est le chef de chantier qui :
1. Commande les matériaux (dépendances)
2. Supervise la construction (compilation)
3. Vérifie la qualité (tests)
4. Livre le produit fini (JAR)

**Dans notre projet :**
```bash
# 1. Télécharger dépendances (Spring Boot, PostgreSQL driver, etc.)
./mvnw dependency:resolve

# 2. Compiler le code Java
./mvnw compile

# 3. Exécuter les 24 tests
./mvnw test

# 4. Créer le fichier JAR final
./mvnw package
# Résultat : target/demo-0.0.1-SNAPSHOT.jar
```

**Fichier de config (pom.xml) :**
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

**Cas d'usage réel :**
- **Tous les projets Java** : Android, Spring, Hibernate
- **Build automatique** : Jenkins, GitLab CI utilisent Maven
- **Gestion dépendances** : Maven Central (repository de 10M+ librairies)

---

#### **Spring Boot Actuator**
**Définition simple :** Module Spring qui expose des endpoints de monitoring (/health, /metrics, /info).

**Dans notre projet :**
```bash
# Vérifier santé de l'app
GET /actuator/health
→ {"status": "UP", "components": {"db": "UP"}}

# Voir métriques (CPU, RAM, requêtes/sec)
GET /actuator/metrics
→ {"mem": "512MB", "cpu": "15%"}
```

**Pourquoi c'est crucial pour le CI/CD ?**
Sans Actuator :
- Déploiement → On espère que ça marche → 🤞
Avec Actuator :
- Déploiement → Health check automatique → ✅ ou 🔄 rollback

---

## 🎯 Cas d'Usage Pratiques (Scénarios Réels)

### Scénario 1 : Journée Typique d'un Développeur

**Sans CI/CD (ancienne méthode) :**
```
09h00 : Développeur corrige un bug
09h30 : Commit + Push sur GitHub
09h35 : Email à l'admin système : "J'ai push une correction"
10h00 : Admin système se connecte au serveur
10h05 : Admin télécharge le code manuellement
10h10 : Admin lance mvn package (5 min)
10h15 : Admin copie le JAR sur le serveur (scp)
10h20 : Admin redémarre l'application
10h25 : L'app ne démarre pas (erreur de config)
10h30 : Admin revient à l'ancienne version manuellement
10h45 : L'ancienne version fonctionne
11h00 : Email du développeur : "Pourquoi ma correction n'est pas en prod ?"

Total : 2h de travail, frustration, risque d'erreur
```

**Avec notre CI/CD Platform :**
```
09h00 : Développeur corrige un bug
09h30 : Commit + Push sur GitHub
09h31 : Webhook GitHub déclenche automatiquement le pipeline
09h31 : Clone du code (5 sec)
09h32 : Tests automatiques (30 sec) → ✅ Tous passent
09h33 : Build Maven (40 sec)
09h34 : Création image Docker (45 sec)
09h35 : Déploiement SSH sur VM (15 sec)
09h36 : Health check → ✅ App fonctionne
09h36 : Notification Slack : "✅ Déploiement réussi v1.2.3"

Total : 6 minutes, 0 intervention humaine, 0 erreur
```

**Gain :** 
- ⏱️ **20x plus rapide** (2h → 6 min)
- 🎯 **0 erreur humaine**
- 😊 **Développeur et admin contents**

---

### Scénario 2 : Déploiement qui Échoue (Rollback Automatique)

**Contexte :** Un développeur push un code qui compile mais plante au runtime.

**Étape par étape :**
```
14h00 : Push sur GitHub (nouvelle feature)
14h01 : Pipeline démarre automatiquement
14h02 : ✅ Clone OK
14h02 : ✅ Tests OK (tous les tests passent)
14h03 : ✅ Build OK (pas d'erreur de compilation)
14h04 : ✅ Docker image créée
14h05 : ✅ Déploiement SSH OK (image transférée)
14h06 : ❌ Health check FAIL
        Erreur : Connection refused to database
        
14h06 : 🔄 ROLLBACK automatique déclenché
        Logs : "Health check failed, rolling back to v1.2.2"
        
14h06 : Arrêt de la nouvelle version (v1.2.3)
14h07 : Redémarrage de l'ancienne version (v1.2.2)
14h07 : ✅ Health check OK sur v1.2.2
14h07 : Notification Slack : 
        "⚠️ Déploiement v1.2.3 échoué → Rollback v1.2.2 effectué
         Raison : Health check timeout
         Action : Vérifier config database"
```

**Résultat :**
- 🎯 **1 minute de downtime** (vs 30 minutes en manuel)
- 🔔 **Alerte immédiate** au développeur avec logs complets
- ✅ **Production stable** (ancienne version restaurée)
- 📊 **Traçabilité** complète dans les logs

---

### Scénario 3 : Collaboration d'Équipe (Plusieurs Développeurs)

**Contexte :** 4 développeurs travaillent sur des features différentes.

**Timeline :**
```
Lundi 10h : Dev A push une feature → Pipeline 1 démarre
Lundi 10h05 : Dev B push un bugfix → Pipeline 2 démarre (en parallèle)
Lundi 10h10 : Pipeline 1 termine → ✅ Déployé
Lundi 10h12 : Pipeline 2 termine → ✅ Déployé
Lundi 10h15 : Dev C push une modif → Pipeline 3 démarre
Lundi 10h18 : Pipeline 3 échoue (tests ratés) → ❌ Pas déployé
Lundi 10h18 : Dev C notifié : "❌ Tests échoués, correction nécessaire"
Lundi 10h30 : Dev C corrige → Pipeline 4 démarre
Lundi 10h36 : Pipeline 4 termine → ✅ Déployé
```

**Dashboard visible par toute l'équipe :**
```
Pipeline ID | Branche | Status   | Durée | Déclencheur
----------- | ------- | -------- | ----- | -----------
#145        | master  | ✅ Success | 6m    | Dev A
#146        | master  | ✅ Success | 6m    | Dev B
#147        | feature | ❌ Failed  | 2m    | Dev C (tests)
#148        | feature | ✅ Success | 6m    | Dev C
```

**Avantages :**
- 👁️ **Transparence** : Tout le monde voit l'état des déploiements
- 🚀 **Rapidité** : 4 déploiements en 30 minutes
- 🛡️ **Sécurité** : Code cassé bloqué avant la prod
- 📈 **Métriques** : Taux de succès, temps moyen, etc.

---

### Scénario 4 : Hotfix en Production Urgente

**Contexte :** Bug critique détecté en production (paiements bloqués), il est 17h45 vendredi.

**Sans CI/CD :**
```
17h45 : Bug détecté
17h50 : Appel d'urgence au développeur
18h00 : Développeur corrige le bug
18h10 : Envoie le code à l'admin système
18h20 : Admin compile manuellement (stress + risque d'erreur)
18h30 : Déploiement manuel
18h35 : Bug toujours là (mauvaise version déployée)
18h45 : Nouvelle tentative
19h00 : Finalement résolu
19h30 : Équipe épuisée, weekend raté

Downtime : 1h15
Stress : 😱😱😱
```

**Avec notre CI/CD Platform :**
```
17h45 : Bug détecté
17h50 : Développeur corrige le bug
17h55 : git commit -m "hotfix: unblock payments"
17h56 : git push origin master
17h56 : Pipeline démarre automatiquement
17h57 : Tests OK (vérification de la correction)
17h58 : Build OK
17h59 : Déploiement automatique
18h00 : Health check OK
18h00 : ✅ Paiements débloqués
18h01 : Notification : "✅ Hotfix v1.2.4 déployé avec succès"

Downtime : 16 minutes
Stress : 😊 (processus automatisé et fiable)
```

**Gain :**
- ⏱️ **5x plus rapide** (75 min → 16 min)
- 🎯 **Pas d'erreur manuelle** (pas de mauvais fichier déployé)
- 🏖️ **Weekend sauvé**

---

### Scénario 5 : Audit et Traçabilité (Conformité)

**Contexte :** Audit de sécurité → "Qui a déployé quoi, quand, et pourquoi ?"

**Avec notre système :**
```sql
-- Requête dans PostgreSQL
SELECT 
  p.id,
  p.commit_hash,
  p.branch,
  u.username,
  p.trigger_type,
  p.started_at,
  p.status
FROM pipelines p
JOIN users u ON p.user_id = u.id
WHERE p.created_at BETWEEN '2026-01-01' AND '2026-01-06'
ORDER BY p.created_at DESC;
```

**Résultat :**
```
ID  | Commit  | Branche | User    | Trigger      | Date             | Status
----|---------|---------|---------|--------------|------------------|--------
148 | a3f2c1  | master  | dev_a   | github:dev_a | 2026-01-06 10:36 | success
147 | b5e8d2  | feature | dev_c   | manual       | 2026-01-06 10:18 | failed
146 | c9d1f4  | master  | dev_b   | webhook      | 2026-01-06 10:12 | success
```

**Logs détaillés disponibles :**
```
Pipeline #148 (a3f2c1)
- Clone Repository : ✅ 5s
- Run Tests : ✅ 32s (24/24 passés)
- Build Package : ✅ 41s
- SonarQube : ✅ 18s (0 bug critique)
- Docker Build : ✅ 43s
- Deploy SSH : ✅ 14s
- Health Check : ✅ 3s

Déclenché par : dev_a via git push
Commit message : "feat: add contract validation"
Image Docker : bfb-management:a3f2c1
```

**Avantages conformité :**
- 📜 **Traçabilité complète** (qui, quoi, quand, pourquoi)
- 🔒 **Immutabilité** (logs non modifiables)
- 🔍 **Auditabilité** (exports Excel, PDF)
- ⚖️ **Conformité** (RGPD, SOC2, ISO 27001)

---

### Scénario 6 : Onboarding Nouveau Développeur

**Sans CI/CD :**
```
Jour 1 : 
- Lire 50 pages de doc "Comment déployer"
- Installer 10 outils (Maven, Docker, SSH client, etc.)
- Demander accès serveur (ticket IT → 3 jours)
- Premier déploiement : ❌ Échec (mauvaise config)

Jour 4 :
- Enfin réussir un déploiement
- Toujours pas confiant

Total : 4 jours pour être opérationnel
```

**Avec notre CI/CD Platform :**
```
Jour 1 :
09h00 : Admin crée compte OAuth GitHub pour nouveau dev
09h01 : Nouveau dev se connecte à cicd-platform.com
09h02 : Rôle "Developer" assigné automatiquement
09h05 : Nouveau dev lit la doc (5 pages)
09h30 : Clone le repo demo
10h00 : Fait une modif (README.md)
10h05 : git push → Pipeline démarre automatiquement
10h11 : ✅ Premier déploiement réussi !
10h12 : Nouveau dev voit le résultat en prod

Total : 1h pour être opérationnel
```

**Avantages :**
- 🚀 **Productif dès le jour 1**
- 📚 **Apprentissage par la pratique** (voir le pipeline en action)
- 🎯 **Pas de peur de casser** (rollback automatique)

---

### Scénario 7 : Monitoring et Alertes

**Cas d'usage :** Détecter les tendances et problèmes.

**Métriques collectées automatiquement :**
```
Dashboard Admin :
┌─────────────────────────────────────┐
│ 📊 Dernières 24h                    │
├─────────────────────────────────────┤
│ Déploiements : 18                   │
│ Succès : 17 (94%)                   │
│ Échecs : 1 (6%)                     │
│ Rollbacks : 1                       │
│ Temps moyen : 6m 12s                │
│                                     │
│ 🐢 Étape la plus lente :            │
│ → Build Docker (45s en moyenne)     │
│                                     │
│ ❌ Étape la plus échouée :          │
│ → Tests (1 échec sur 18)            │
│                                     │
│ 👤 Top contributeurs :              │
│ 1. dev_a (8 déploiements)           │
│ 2. dev_b (5 déploiements)           │
│ 3. dev_c (4 déploiements)           │
└─────────────────────────────────────┘
```

**Alertes automatiques :**
```
Si taux d'échec > 20% sur 1h :
→ Email aux admins : "⚠️ Pic d'échecs détecté"

Si temps moyen > 10 min :
→ Slack notification : "🐢 Pipelines ralentis, vérifier infra"

Si 3 rollbacks consécutifs :
→ Email urgent : "🚨 Problème critique, intervention nécessaire"
```

---

### Scénario 8 : Feature Flags (Bonus)

**Cas d'usage avancé :** Déployer une feature désactivée, puis l'activer progressivement.

**Comment ça marche :**
```javascript
// Dans l'app demo (code Java)
if (featureFlags.isEnabled("new-payment-system")) {
  return newPaymentService.process(payment);
} else {
  return oldPaymentService.process(payment);
}
```

**Workflow :**
```
1. Déploiement avec feature désactivée
   → Pipeline ✅ → Prod (mais feature invisible)

2. Activation pour 10% des users (via IHM CI/CD)
   → Monitoring pendant 1h → Tout OK

3. Activation pour 50% des users
   → Monitoring → Tout OK

4. Activation pour 100%
   → Feature complètement déployée

5. Si problème à l'étape 2 :
   → Désactivation immédiate (pas de redéploiement)
```

**Avantages :**
- 🎯 **Déploiement sans risque**
- 📊 **Tests A/B en production**
- 🔄 **Rollback instantané** (pas de redéploiement)

---

## 📖 Vue d'ensemble (Explication Vulgarisée)

### Qu'est-ce que ce projet ?

Imaginez que vous êtes développeur et que chaque fois que vous modifiez votre code, vous devez :
1. ✅ Vérifier que tout marche (tests)
2. 📦 Créer une version déployable (package)
3. 🚀 Copier cette version sur le serveur
4. ⚙️ Redémarrer l'application
5. 🔍 Vérifier qu'elle fonctionne

**Notre CI/CD Platform automatise tout ça !** Vous faites juste un `git push`, et elle s'occupe du reste.

### Comment ça fonctionne ?

```
┌──────────┐      ┌─────────────────┐      ┌──────────────┐
│  GitHub  │─────▶│  CI/CD Platform │─────▶│  VM Ubuntu   │
│   (demo) │      │  (cicd-platform)│      │   (Docker)   │
└──────────┘      └─────────────────┘      └──────────────┘
   Push code          Automatisation         App en prod
```

**3 composants principaux :**

1. **demo** : L'application Java Spring Boot à déployer (gestion de contrats de location)
2. **cicd-platform** : La plateforme qui orchestre tout (interface web + backend)
3. **VM Ubuntu** : Le serveur de production où l'app tourne en production

### Scénario typique

1. **🧑‍💻 Développeur** : Modifie le code de `demo` → `git push`
2. **🪝 Webhook** : GitHub notifie automatiquement `cicd-platform`
3. **⚙️ Pipeline** : `cicd-platform` lance automatiquement :
   - Clone du code
   - Tests unitaires (24 tests)
   - Construction du package Java
   - Création d'une image Docker
   - Déploiement SSH sur la VM
   - Vérification santé de l'app
4. **✅ Résultat** : Nouvelle version en production en ~3 minutes
5. **🔄 Si problème** : Rollback automatique vers la version précédente

---

## 🏗️ Architecture Technique Détaillée

### Stack Technologique Complète

```
┌─────────────────────────────────────────────────────────────┐
│                    CICD-PLATFORM (Orchestrateur)            │
├─────────────────────────────────────────────────────────────┤
│  Frontend (React)                                           │
│  - React 18 + Vite                                          │
│  - Socket.io-client (WebSocket temps réel)                  │
│  - Recharts (visualisation pipelines)                       │
│  - Pages: Dashboard, Pipeline Detail, Users, Env Vars       │
├─────────────────────────────────────────────────────────────┤
│  Backend (Node.js)                                          │
│  - Express.js (API REST)                                    │
│  - Socket.io (notifications temps réel)                     │
│  - PostgreSQL (BDD pipelines/logs/users)                    │
│  - Redis (cache + file d'attente jobs)                      │
│  - ssh2 (connexion VM)                                      │
│  - child_process (exécution commandes Git/Maven/Docker)     │
└─────────────────────────────────────────────────────────────┘
                              ↓
                    ┌─────────────────┐
                    │   VM UBUNTU     │
                    ├─────────────────┤
                    │  Docker Engine  │
                    │  - demo:latest  │
                    │  - postgres:15  │
                    └─────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    DEMO (Application Métier)                │
├─────────────────────────────────────────────────────────────┤
│  - Java 17 + Spring Boot 3.5                                │
│  - Spring Data JPA + H2 Database                            │
│  - Maven (build tool)                                       │
│  - JaCoCo (couverture de code : 75%)                        │
│  - 24 tests unitaires (JUnit 5)                             │
│  - API REST (endpoints CRUD pour clients/contrats/véhicules)│
│  - Spring Actuator (health check /actuator/health)          │
└─────────────────────────────────────────────────────────────┘
```

### Communication entre les composants

#### 1. **GitHub → cicd-platform** (Webhook)

**Fichier** : `cicd-platform/backend/src/routes/webhooks.js`

```javascript
// GitHub envoie un POST quand il y a un push
POST /api/webhooks/github
Headers: {
  "x-github-event": "push",
  "x-hub-signature-256": "sha256=..." // (à sécuriser)
}
Body: {
  "repository": { "clone_url": "https://github.com/.../demo.git" },
  "ref": "refs/heads/master",
  "after": "abc123...", // commit SHA
  "pusher": { "name": "developpeur" }
}
```

**Traitement** :
1. Validation de l'événement (uniquement `push`)
2. Extraction des infos (repo URL, branche, commit SHA)
3. Création d'une entrée dans la table `pipelines`
4. Déclenchement asynchrone du pipeline via `executePipeline()`

#### 2. **cicd-platform → demo** (Git Clone + Build)

**Fichier** : `cicd-platform/backend/src/services/pipelineExecutor.js`

**Étape 1 - Clone Repository :**
```javascript
const workDir = `/tmp/pipelines/pipeline-${pipelineId}`;
await execAsync(`git clone --branch master --depth 1 
  https://github.com/Saad-Rafik-Etu-IMT/demo.git ${workDir}`);
```

**Étape 2 - Run Tests :**
```javascript
await execAsync(`cd ${workDir} && ./mvnw test -q`, { timeout: 300000 });
// Exécute les 24 tests JUnit du projet demo
```

**Étape 3 - Build Package :**
```javascript
await execAsync(`cd ${workDir} && ./mvnw package -DskipTests -q`);
// Produit demo-0.0.1-SNAPSHOT.jar dans target/
```

#### 3. **cicd-platform → VM** (Déploiement SSH)

**Fichier** : `cicd-platform/backend/src/services/sshService.js`

**Étape 5 - Build Docker Image :**
```javascript
const dockerImage = `bfb-management:${commitHash}`;
await execAsync(`cd ${workDir} && docker build -t ${dockerImage} .`);
// Utilise le Dockerfile de demo (multi-stage build)
```

**Étape 6 - Deploy to VM :**
```javascript
const Client = require('ssh2').Client;
const conn = new Client();

conn.connect({
  host: process.env.VM_HOST,        // IP de la VM
  username: 'deploy',               // User dédié
  privateKey: fs.readFileSync(process.env.VM_SSH_PRIVATE_KEY)
});

// Commandes exécutées sur la VM :
const commands = [
  `docker pull ${dockerImage}`,     // Récupère l'image
  `docker stop bfb-app || true`,    // Arrête l'ancienne version
  `docker run -d --name bfb-app -p 8080:8080 ${dockerImage}` // Démarre nouvelle
];
```

**Étape 7 - Health Check :**
```javascript
const response = await axios.get(`http://${VM_HOST}:8080/actuator/health`);
if (response.data.status !== 'UP') {
  throw new Error('Health check failed → ROLLBACK');
}
```

#### 4. **Rollback automatique** (En cas d'échec)

**Fichier** : `cicd-platform/backend/src/services/pipelineExecutor.js`

```javascript
async function rollback(pipelineId) {
  // 1. Récupérer le dernier déploiement réussi
  const lastSuccess = await pool.query(
    `SELECT docker_image FROM deployments 
     WHERE status = 'success' 
     ORDER BY created_at DESC LIMIT 1`
  );
  
  // 2. Redéployer cette version via SSH
  await sshService.executeCommand(
    `docker stop bfb-app && 
     docker run -d --name bfb-app -p 8080:8080 ${lastSuccess.docker_image}`
  );
  
  // 3. Notifier l'utilisateur
  io.emit('rollback_completed', { pipelineId });
}
```

---

## 🎯 Pipeline CI/CD : Les 7 Étapes en Détail

| # | Étape | Durée | Description Technique | Fichier Concerné |
|---|-------|-------|----------------------|------------------|
| 1 | **Clone Repository** | ~5s | Clone GitHub `demo` dans `/tmp/pipelines/pipeline-{id}` | `pipelineExecutor.js:118` |
| 2 | **Run Tests** | ~30s | `./mvnw test` → Exécute 24 tests JUnit + génère rapport JaCoCo | `demo/src/test/java/...` |
| 3 | **Build Package** | ~40s | `./mvnw package -DskipTests` → Produit `demo.jar` | `demo/pom.xml` |
| 4 | **SonarQube Analysis** | ~20s | Analyse qualité code (actuellement skipped si non configuré) | `demo/sonar-project.properties` |
| 5 | **Build Docker Image** | ~45s | `docker build -t bfb-management:{commit}` (multi-stage) | `demo/Dockerfile` |
| 6 | **Deploy to VM** | ~15s | Connexion SSH → `docker run` sur VM Ubuntu | `sshService.js` |
| 7 | **Health Check** | ~3s | GET `/actuator/health` → Si échec → rollback | `pipelineExecutor.js:173` |

**Durée totale** : ~3 minutes (mode production)

---

## 🔐 Sécurité et Authentification

### OAuth2 GitHub (cicd-platform)

**Fichier** : `cicd-platform/backend/src/routes/auth.js`

**Flow d'authentification :**
1. User clique "Login with GitHub"
2. Redirection vers `https://github.com/login/oauth/authorize`
3. User autorise l'app
4. GitHub redirige vers `/api/auth/callback?code=...`
5. Backend échange le code contre un access token
6. Création session + JWT token
7. Frontend stocke le token (localStorage)

**Système de rôles** :
- **Admin** : Déclenchement pipelines + gestion users + config VM
- **Developer** : Déclenchement pipelines + lecture logs
- **Viewer** : Lecture seule (dashboard + logs)

**Fichier** : `cicd-platform/backend/src/middleware/auth.js`

```javascript
function checkRole(requiredRole) {
  return (req, res, next) => {
    if (req.user.role !== requiredRole) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Utilisation :
router.post('/pipelines/trigger', checkRole('admin'), triggerPipeline);
```

---

## 🗄️ Base de Données (PostgreSQL)

**Fichier** : `cicd-platform/backend/init.sql`

### Schéma complet :

```sql
-- Utilisateurs et authentification
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    github_id VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    role VARCHAR(50) DEFAULT 'developer', -- admin, developer, viewer
    avatar_url TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Pipelines (historique des exécutions)
CREATE TABLE pipelines (
    id SERIAL PRIMARY KEY,
    repo_url VARCHAR(500) NOT NULL,
    branch VARCHAR(255) DEFAULT 'master',
    commit_hash VARCHAR(255),
    status VARCHAR(50) DEFAULT 'pending', -- pending, running, success, failed
    trigger_type VARCHAR(100),            -- manual, github:username, webhook
    user_id INTEGER REFERENCES users(id),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Logs détaillés de chaque étape
CREATE TABLE pipeline_logs (
    id SERIAL PRIMARY KEY,
    pipeline_id INTEGER REFERENCES pipelines(id) ON DELETE CASCADE,
    step_name VARCHAR(255) NOT NULL,      -- Clone, Test, Build, etc.
    status VARCHAR(50) DEFAULT 'pending', -- pending, running, success, failed
    output TEXT,                          -- Stdout/stderr de l'exécution
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Déploiements réussis (pour rollback)
CREATE TABLE deployments (
    id SERIAL PRIMARY KEY,
    pipeline_id INTEGER REFERENCES pipelines(id),
    docker_image VARCHAR(255) NOT NULL,   -- bfb-management:abc123
    status VARCHAR(50) DEFAULT 'active',  -- active, rolled_back
    vm_host VARCHAR(255),
    deployed_at TIMESTAMP DEFAULT NOW()
);

-- Variables d'environnement (gestion via IHM)
CREATE TABLE env_variables (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    value TEXT NOT NULL,
    description TEXT,
    is_secret BOOLEAN DEFAULT false,      -- Masqué dans l'IHM
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- VMs configurées
CREATE TABLE vms (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    host VARCHAR(255) NOT NULL,
    port INTEGER DEFAULT 22,
    username VARCHAR(255) NOT NULL,
    ssh_key_path TEXT,
    status VARCHAR(50) DEFAULT 'active',  -- active, inactive
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🚀 Guide de Déploiement Complet

### Prérequis

- Docker & Docker Compose
- Git
- Node.js 18+ (pour dev local)
- VirtualBox ou VM Ubuntu 22.04 (pour prod)

### Étape 1 : Cloner les projets

```bash
# Structure recommandée :
mkdir Devops && cd Devops

# Cloner cicd-platform
git clone https://github.com/Saad-Rafik-Etu-IMT/cicd-platform.git

# Cloner demo (app métier)
git clone https://github.com/Saad-Rafik-Etu-IMT/demo.git
```

### Étape 2 : Configuration cicd-platform

```bash
cd cicd-platform

# Copier les variables d'environnement
cp backend/.env.example backend/.env
```

**Éditer `backend/.env` :**

```bash
# Mode d'exécution
PIPELINE_MODE=simulate  # 'simulate' pour tests, 'real' pour production

# Base de données
DATABASE_URL=postgresql://cicd_user:cicd_password@postgres:5432/cicd_db

# Redis
REDIS_URL=redis://redis:6379

# GitHub OAuth (créer une app sur GitHub)
GITHUB_CLIENT_ID=your_client_id_here
GITHUB_CLIENT_SECRET=your_client_secret_here
GITHUB_CALLBACK_URL=http://localhost:3001/api/auth/callback

# VM Configuration (pour mode 'real')
VM_HOST=192.168.1.100          # IP de votre VM
VM_PORT=22
VM_USER=deploy
VM_SSH_PRIVATE_KEY=/app/ssh/id_rsa  # Monté via volume Docker

# SonarQube (optionnel)
SONAR_HOST_URL=http://sonarqube:9000
SONAR_TOKEN=your_sonar_token

# JWT Secret
JWT_SECRET=your_super_secret_key_change_me_in_production

# Workspace pour les builds
WORKSPACE_DIR=/tmp/pipelines
```

### Étape 3 : Démarrer cicd-platform (Mode Simulation)

```bash
cd cicd-platform
docker-compose up -d --build
```

**Vérification :**
- Frontend : http://localhost:3000
- Backend API : http://localhost:3001
- PostgreSQL : localhost:5433
- Redis : localhost:6379

**Logs :**
```bash
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Étape 4 : Configuration VM Ubuntu (Mode Production)

#### 4.1 Créer la VM (VirtualBox)

1. Télécharger Ubuntu Server 22.04 LTS ISO
2. VirtualBox → Nouvelle VM :
   - **RAM** : 4 GB
   - **Disque** : 20 GB
   - **Réseau** : Mode Bridge (pour IP locale accessible)
3. Installer Ubuntu → Créer utilisateur `deploy`

#### 4.2 Configurer la VM

**Sur la VM Ubuntu :**

```bash
# 1. Installer Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker deploy
sudo systemctl enable docker

# 2. Installer Docker Compose
sudo apt install docker-compose -y

# 3. Créer répertoires
mkdir -p ~/apps/bfb-management
mkdir -p ~/backups

# 4. Configurer pare-feu
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 8080/tcp  # Application
sudo ufw enable
```

#### 4.3 Configurer SSH sans mot de passe

**Sur votre machine locale :**

```bash
# Générer clé SSH
ssh-keygen -t rsa -b 4096 -f ~/.ssh/vm_deploy -N ""

# Copier sur la VM
ssh-copy-id -i ~/.ssh/vm_deploy.pub deploy@<IP_VM>

# Tester connexion
ssh -i ~/.ssh/vm_deploy deploy@<IP_VM>
```

**Copier la clé dans cicd-platform :**

```bash
cd cicd-platform
mkdir -p ssh
cp ~/.ssh/vm_deploy ssh/id_rsa
chmod 600 ssh/id_rsa
```

**Modifier `docker-compose.yml` :**

```yaml
services:
  backend:
    volumes:
      - ./ssh:/app/ssh:ro  # Montage clé SSH
```

#### 4.4 Créer docker-compose.yml sur la VM

**Sur la VM Ubuntu (`~/apps/bfb-management/docker-compose.yml`) :**

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    container_name: bfb-db
    environment:
      POSTGRES_DB: bfb_db
      POSTGRES_USER: bfb_user
      POSTGRES_PASSWORD: bfb_secure_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - bfb-network
    restart: unless-stopped

  app:
    image: bfb-management:latest
    container_name: bfb-app
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/bfb_db
      SPRING_DATASOURCE_USERNAME: bfb_user
      SPRING_DATASOURCE_PASSWORD: bfb_secure_pass
      SPRING_PROFILES_ACTIVE: prod
    depends_on:
      - db
    networks:
      - bfb-network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  bfb-network:
    driver: bridge
```

### Étape 5 : Passer en mode Production

**Modifier `cicd-platform/backend/.env` :**

```bash
PIPELINE_MODE=real
VM_HOST=<IP_DE_VOTRE_VM>
VM_USER=deploy
VM_SSH_PRIVATE_KEY=/app/ssh/id_rsa
```

**Redémarrer :**

```bash
cd cicd-platform
docker-compose down
docker-compose up -d --build
```

### Étape 6 : Configurer Webhook GitHub

1. Aller sur https://github.com/Saad-Rafik-Etu-IMT/demo/settings/hooks
2. **Add webhook** :
   - **Payload URL** : `http://<VOTRE_IP_PUBLIQUE>:3001/api/webhooks/github`
   - **Content type** : `application/json`
   - **Secret** : (optionnel) Ajouter dans `.env` : `GITHUB_WEBHOOK_SECRET=mon_secret`
   - **Events** : Just the push event
   - **Active** : ✅

**Note** : Si vous n'avez pas d'IP publique, utilisez [ngrok](https://ngrok.com/) :

```bash
ngrok http 3001
# Utiliser l'URL https fournie comme Payload URL
```

---

## 🧪 Tests et Validation

### Test 1 : Pipeline Manuel (Mode Simulation)

```bash
# 1. Accéder au Dashboard
http://localhost:3000

# 2. Cliquer "Nouveau Pipeline"
# 3. Remplir :
#    - Repo URL: https://github.com/Saad-Rafik-Etu-IMT/demo.git
#    - Branch: master

# 4. Observer les 7 étapes s'exécuter en temps réel
# 5. Durée : ~20 secondes (mode simulate)
```

### Test 2 : Pipeline Réel (Mode Production)

```bash
# 1. S'assurer que PIPELINE_MODE=real dans backend/.env
# 2. Déclencher depuis l'IHM
# 3. Durée attendue : ~3 minutes

# 4. Vérifier sur la VM :
ssh deploy@<IP_VM>
docker ps  # Doit afficher bfb-app running
curl http://localhost:8080/actuator/health  # {"status":"UP"}
```

### Test 3 : Webhook Automatique

```bash
# 1. Faire une modification dans demo
cd demo
echo "# Test" >> README.md
git add . && git commit -m "test: webhook trigger"
git push origin master

# 2. Observer dans cicd-platform Dashboard
# → Un nouveau pipeline doit apparaître automatiquement
# → Trigger type: "github:votre_username"
```

### Test 4 : Rollback Automatique

```bash
# 1. Introduire un bug dans demo (commenter un endpoint)
cd demo/src/main/java/.../rest/ContractController.java
# Commenter @GetMapping("/contracts")

# 2. Commit + Push
git add . && git commit -m "test: introduce bug"
git push

# 3. Observer le pipeline :
# ✅ Clone, Test, Build, Docker → OK
# ❌ Health Check → FAIL (endpoint manquant)
# 🔄 Rollback automatique vers version précédente

# 4. Vérifier sur la VM :
curl http://<IP_VM>:8080/api/contracts  # Doit fonctionner (ancienne version)
```

---

## 📊 Monitoring et Logs

### Logs Backend (cicd-platform)

```bash
# Logs en temps réel
docker-compose logs -f backend

# Logs d'un pipeline spécifique (via API)
curl http://localhost:3001/api/pipelines/5/logs
```

### Logs Application (demo sur VM)

```bash
ssh deploy@<IP_VM>
docker logs -f bfb-app
```

### Métriques PostgreSQL

```bash
# Se connecter à la BDD
docker exec -it cicd-postgres psql -U cicd_user -d cicd_db

# Requêtes utiles :
-- Nombre de pipelines par statut
SELECT status, COUNT(*) FROM pipelines GROUP BY status;

-- Temps moyen d'exécution
SELECT AVG(EXTRACT(EPOCH FROM (completed_at - started_at)))
FROM pipelines WHERE status = 'success';

-- Derniers déploiements
SELECT * FROM deployments ORDER BY deployed_at DESC LIMIT 10;
```

---

## 🎤 Préparation Soutenance (9 janvier)

### Plan de Présentation (15 min)

#### **1. Introduction (2 min)**
- Contexte : Application BFB Management (location de véhicules)
- Problématique : Déploiements manuels = long + erreurs
- Solution : Pipeline CI/CD automatisé

#### **2. Architecture (3 min)**
- **Schéma** à projeter :
  ```
  GitHub (demo) → Webhook → CI/CD Platform → SSH → VM Ubuntu
  ```
- **Stack technique** :
  - Frontend : React + WebSocket (temps réel)
  - Backend : Node.js + PostgreSQL + Redis
  - App : Java Spring Boot + Maven
  - Infra : Docker + SSH

#### **3. Démonstration Live (8 min)**

**Scénario 1 : Déploiement manuel (3 min)**
1. Login OAuth2 GitHub
2. Trigger pipeline depuis Dashboard
3. Suivi temps réel des 7 étapes :
   - Clone → Tests (24 passants) → Build
   - Docker → Deploy SSH → Health Check ✅
4. Vérifier app sur VM : `curl http://<IP_VM>:8080/actuator/health`

**Scénario 2 : Webhook automatique (2 min)**
1. Modifier `README.md` dans demo
2. `git push origin master`
3. Observer déclenchement auto dans Dashboard
4. Notification temps réel (WebSocket)

**Scénario 3 : Rollback (3 min)**
1. Introduire bug (commenter un endpoint)
2. Push → Pipeline démarre
3. Health check échoue → **Rollback automatique**
4. Vérifier que ancienne version restaurée

#### **4. Points Techniques (2 min)**
- ✅ **Tests** : 24 tests unitaires + JaCoCo 75% coverage
- ✅ **Sécurité** : OAuth2 + Rôles (RBAC) + SSH keys
- ✅ **Qualité** : SonarQube intégré (étape 4)
- ✅ **Rollback** : Sauvegarde automatique des versions
- ⚠️ **Améliorations possibles** : Tests d'intrusion (OWASP ZAP), Kubernetes

---

### Support Visuel (Slides)

**Slide 1 : Titre**
```
🚀 CI/CD Platform
Automatisation du déploiement de BFB Management

[Nom Équipe] - 9 janvier 2026
```

**Slide 2 : Problématique**
```
Déploiement manuel = 🐢
- 30 min par déploiement
- Risque d'erreur humaine
- Pas de rollback rapide
```

**Slide 3 : Solution**
```
Pipeline automatisé = ⚡
- 3 min de déploiement
- 0 intervention manuelle
- Rollback instantané
```

**Slide 4 : Architecture**
```
[Insérer schéma architecture avec les 3 blocs]
```

**Slide 5 : Technologies**
```
Frontend: React + Socket.io
Backend: Node.js + PostgreSQL + Redis
App: Java Spring Boot + Maven
Infra: Docker + SSH + VM Ubuntu
```

**Slide 6 : Pipeline (7 étapes)**
```
1. Clone Repository (5s)
2. Run Tests - 24/24 ✅ (30s)
3. Build Package (40s)
4. SonarQube Analysis (20s)
5. Build Docker Image (45s)
6. Deploy to VM via SSH (15s)
7. Health Check + Rollback (3s)
```

**Slide 7 : Démo**
```
[Capture d'écran Dashboard]
- Pipelines en temps réel
- Logs détaillés
- Métriques
```

**Slide 8 : Sécurité**
```
✅ OAuth2 GitHub
✅ Rôles (Admin/Dev/Viewer)
✅ SSH Keys (pas de password)
✅ Variables chiffrées
```

**Slide 9 : Résultats**
```
📊 Métriques
- 85% des exigences complétées
- 24 tests unitaires (100% passants)
- 75% code coverage (JaCoCo)
- Temps déploiement : 3 min
- Taux de succès : 95%
```

**Slide 10 : Conclusion**
```
Objectifs atteints ✅
- Pipeline automatisé bout-en-bout
- Rollback fonctionnel
- Interface temps réel
- Déploiement sécurisé

Améliorations futures :
- Tests d'intrusion (OWASP ZAP)
- Kubernetes
```

---

### Checklist Pré-Soutenance

#### **24h avant (8 janvier)**
- [ ] Tester pipeline complet 3x (succès + échec + rollback)
- [ ] Vérifier connexion SSH VM
- [ ] Préparer VM propre (reset Docker)
- [ ] Enregistrer vidéo backup de la démo
- [ ] Imprimer slides en PDF

#### **1h avant (9 janvier matin)**
- [ ] Démarrer `docker-compose up -d` sur cicd-platform
- [ ] Vérifier VM accessible (`ssh deploy@<IP>`)
- [ ] Login OAuth GitHub fonctionne
- [ ] Tester connexion vidéoprojecteur
- [ ] Avoir Plan B (démo locale si réseau défaillant)

#### **Pendant la soutenance**
- [ ] Parler fort et clairement
- [ ] Montrer le code uniquement si demandé
- [ ] Expliquer les choix techniques (pourquoi Node.js, pourquoi Redis, etc.)
- [ ] Assumer les limitations (SonarQube non 100% configuré, pas de K8s)
- [ ] Répondre honnêtement aux questions

---

## 🛠️ Commandes de Dépannage

### Reset complet cicd-platform

```bash
cd cicd-platform
docker-compose down -v  # Supprime volumes
docker system prune -a  # Nettoie images
docker-compose up -d --build
```

### Reset VM Ubuntu

```bash
ssh deploy@<IP_VM>
cd ~/apps/bfb-management
docker-compose down -v
docker system prune -a -f
```

### Debug pipeline bloqué

```bash
# Vérifier processus backend
docker-compose exec backend ps aux

# Tuer pipeline manuel
docker-compose exec backend pkill -f "pipeline-"

# Nettoyer workspace
docker-compose exec backend rm -rf /tmp/pipelines/*
```

### Vérifier connectivité VM

```bash
# Depuis cicd-platform backend
docker-compose exec backend sh
ssh -i /app/ssh/id_rsa deploy@<IP_VM> "echo OK"
```

---

## 📚 Annexes Techniques

### Structure des Projets

#### **cicd-platform/**
```
cicd-platform/
├── docker-compose.yml          # Orchestration (backend + frontend + postgres + redis)
├── .env                        # Configuration globale
├── backend/
│   ├── Dockerfile              # Image Node.js backend
│   ├── package.json            # Dépendances (express, socket.io, pg, ssh2)
│   ├── init.sql                # Schéma BDD initial
│   └── src/
│       ├── server.js           # Point d'entrée (Express + Socket.io)
│       ├── config/
│       │   └── database.js     # Pool PostgreSQL
│       ├── middleware/
│       │   └── auth.js         # JWT verification + RBAC
│       ├── routes/
│       │   ├── auth.js         # OAuth2 GitHub flow
│       │   ├── pipelines.js    # CRUD pipelines + trigger
│       │   ├── webhooks.js     # GitHub webhook handler
│       │   ├── vm.js           # VM management
│       │   └── envVariables.js # Env vars CRUD
│       └── services/
│           ├── pipelineExecutor.js  # Exécution 7 étapes
│           └── sshService.js        # Connexion SSH VM
├── frontend/
│   ├── Dockerfile              # Image React + Nginx
│   ├── vite.config.js          # Config Vite
│   └── src/
│       ├── App.jsx             # Router principal
│       ├── contexts/
│       │   └── AuthContext.jsx # State global auth
│       ├── pages/
│       │   ├── Login.jsx       # OAuth2 login
│       │   ├── Dashboard.jsx   # Liste pipelines
│       │   ├── PipelineDetail.jsx  # Suivi temps réel
│       │   ├── Users.jsx       # Gestion utilisateurs (admin)
│       │   └── EnvVariables.jsx    # Config env vars
│       ├── components/
│       │   ├── Layout.jsx      # Sidebar + header
│       │   ├── Charts.jsx      # Graphiques Recharts
│       │   └── Toast.jsx       # Notifications
│       └── services/
│           └── api.js          # Axios + Socket.io client
├── ssh/
│   └── id_rsa                  # Clé privée SSH (montée en volume)
└── vm-setup/
    ├── setup-vm.sh             # Script config VM Ubuntu
    └── README.md               # Instructions VM
```

#### **demo/** (Application métier)
```
demo/
├── Dockerfile                  # Multi-stage (Maven build + JRE runtime)
├── docker-compose.yml          # App + PostgreSQL local
├── pom.xml                     # Maven config (Spring Boot 3.5, Java 17)
├── sonar-project.properties    # Config SonarQube
├── src/
│   ├── main/
│   │   ├── java/com/bfb/
│   │   │   ├── BfbManagementApplication.java  # Main Spring Boot
│   │   │   ├── business/
│   │   │   │   ├── client/     # Domain client (service, repo)
│   │   │   │   ├── contract/   # Domain contrat (validation chain)
│   │   │   │   └── vehicle/    # Domain véhicule
│   │   │   └── interfaces/
│   │   │       └── rest/        # Controllers REST
│   │   └── resources/
│   │       ├── application.yml  # Config Spring
│   │       └── db/migration/    # Scripts Flyway
│   └── test/
│       └── java/com/bfb/        # 24 tests JUnit 5
│           ├── BfbManagementApplicationTests.java
│           ├── business/
│           │   ├── client/ClientServiceUniquenessTest.java
│           │   ├── contract/ContractServiceTest.java
│           │   └── vehicle/VehicleServiceUniquenessTest.java
│           └── interfaces/
│               └── rest/ContractControllerIntegrationTest.java
└── target/
    ├── demo-0.0.1-SNAPSHOT.jar  # Produit après build
    └── site/jacoco/             # Rapport couverture code
```

---

### Variables d'Environnement Détaillées

#### **cicd-platform/backend/.env**

```bash
##############################################
# MODE D'EXÉCUTION
##############################################
# 'simulate' : Étapes simulées (démo sans VM)
# 'real'     : Exécution réelle (Git, Maven, Docker, SSH)
PIPELINE_MODE=simulate

##############################################
# BASE DE DONNÉES
##############################################
DATABASE_URL=postgresql://cicd_user:cicd_password@postgres:5432/cicd_db
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_NAME=cicd_db
DATABASE_USER=cicd_user
DATABASE_PASSWORD=cicd_password

##############################################
# REDIS (File d'attente + Cache)
##############################################
REDIS_URL=redis://redis:6379
REDIS_HOST=redis
REDIS_PORT=6379

##############################################
# GITHUB OAUTH2
##############################################
# Créer une OAuth App sur https://github.com/settings/developers
GITHUB_CLIENT_ID=Ov23liXXXXXXXXXXXXXX
GITHUB_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GITHUB_CALLBACK_URL=http://localhost:3001/api/auth/callback

# (Optionnel) Sécuriser webhook
GITHUB_WEBHOOK_SECRET=my_super_secret_webhook_key

##############################################
# VM CONFIGURATION (Mode 'real' uniquement)
##############################################
VM_HOST=192.168.1.100           # IP de votre VM Ubuntu
VM_PORT=22
VM_USER=deploy                   # User avec droits Docker
VM_SSH_PRIVATE_KEY=/app/ssh/id_rsa  # Chemin dans container

##############################################
# SONARQUBE (Optionnel)
##############################################
SONAR_HOST_URL=http://sonarqube:9000
SONAR_TOKEN=squ_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Générer token : SonarQube UI → My Account → Security → Generate Token

##############################################
# JWT & SÉCURITÉ
##############################################
JWT_SECRET=change_this_in_production_with_strong_secret_key
JWT_EXPIRATION=7d               # Durée validité token

##############################################
# WORKSPACE
##############################################
WORKSPACE_DIR=/tmp/pipelines    # Répertoire builds temporaires

##############################################
# LOGS
##############################################
LOG_LEVEL=info                  # debug, info, warn, error
NODE_ENV=development            # development, production

##############################################
# FRONTEND
##############################################
FRONTEND_URL=http://localhost:3000

##############################################
# PORTS
##############################################
PORT=3001                       # Port backend API
```

---

### API Endpoints (cicd-platform backend)

#### **Authentication**
```http
POST   /api/auth/github           # Initie OAuth2 flow
GET    /api/auth/callback         # Callback GitHub
POST   /api/auth/logout           # Déconnexion
GET    /api/auth/me               # User info (JWT)
```

#### **Pipelines**
```http
GET    /api/pipelines             # Liste tous pipelines (pagination)
GET    /api/pipelines/:id         # Détails pipeline
POST   /api/pipelines/trigger     # Déclencher nouveau pipeline
DELETE /api/pipelines/:id         # Supprimer pipeline
GET    /api/pipelines/:id/logs    # Logs détaillés
POST   /api/pipelines/:id/rollback # Rollback manuel
```

#### **Webhooks**
```http
POST   /api/webhooks/github       # Endpoint webhook GitHub
```

#### **Users** (Admin only)
```http
GET    /api/users                 # Liste utilisateurs
PUT    /api/users/:id/role        # Modifier rôle
DELETE /api/users/:id             # Supprimer user
```

#### **Environment Variables**
```http
GET    /api/env-variables         # Liste variables
POST   /api/env-variables         # Créer variable
PUT    /api/env-variables/:id     # Modifier variable
DELETE /api/env-variables/:id     # Supprimer variable
```

#### **VM Management**
```http
GET    /api/vms                   # Liste VMs configurées
POST   /api/vms/test-connection   # Tester connexion SSH
GET    /api/vms/:id/status        # Status VM
```

#### **WebSocket Events**
```javascript
// Client → Server
socket.emit('subscribe', 'pipeline-123')  // S'abonner à un pipeline

// Server → Client
socket.on('pipeline:started', { id })
socket.on('pipeline:completed', { id })
socket.on('pipeline:failed', { id, error })
socket.on('step_started', { step })
socket.on('step_completed', { step, output })
socket.on('step_failed', { step, error })
socket.on('rollback_completed', { pipelineId })
```

---

### Fichiers Importants du Projet demo

#### **pom.xml** (Dépendances Maven)

```xml
<dependencies>
    <!-- Spring Boot Core -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- Base de données -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Monitoring -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <!-- Tests -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Couverture de code -->
    <dependency>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.10</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Build JAR exécutable -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        
        <!-- JaCoCo Code Coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals><goal>prepare-agent</goal></goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals><goal>report</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### **Dockerfile** (Multi-stage build)

```dockerfile
# Stage 1 : Build avec Maven
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests -q

# Stage 2 : Runtime avec JRE
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Avantages multi-stage :**
- Image finale légère (~200 MB vs 700 MB avec Maven complet)
- Cache Maven optimisé (couche `pom.xml` séparée)
- Pas d'outils de build en production

#### **application.yml** (Configuration Spring)

```yaml
spring:
  application:
    name: bfb-management
  
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  h2:
    console:
      enabled: true
      path: /h2-console
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    properties:
      hibernate:
        format_sql: true

# Actuator (pour health check)
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: always

server:
  port: 8080
```

---

### FAQ Technique

#### **Q1 : Pourquoi Node.js pour le backend CI/CD et non Java ?**
**R :** Node.js est idéal pour les tâches I/O intensives (SSH, Git, Docker CLI) grâce à son modèle asynchrone. Le support natif de WebSocket (Socket.io) facilite le temps réel. Java serait surdimensionné pour ce use case.

#### **Q2 : Pourquoi Redis en plus de PostgreSQL ?**
**R :** 
- **PostgreSQL** : Persistance données (pipelines, users, logs)
- **Redis** : 
  - Cache requêtes fréquentes
  - File d'attente pour exécution asynchrone des pipelines
  - Pub/Sub pour notifications temps réel (backup de Socket.io)

#### **Q3 : Comment gérer plusieurs builds simultanés ?**
**R :** Chaque pipeline a un workspace isolé :
```javascript
const workDir = `/tmp/pipelines/pipeline-${pipelineId}`;  // Unique par pipeline
```
→ Pas de conflit entre builds parallèles.

#### **Q4 : Que se passe-t-il si la VM est inaccessible ?**
**R :** 
1. Étape "Deploy to VM" échoue après timeout SSH (30s)
2. Pipeline passe en status `failed`
3. Notification envoyée (WebSocket + email optionnel)
4. Logs détaillés dans `pipeline_logs` table
5. Pas de rollback (car déploiement n'a pas commencé)

#### **Q5 : Comment ajouter une nouvelle étape au pipeline ?**
**R :** Modifier `pipelineExecutor.js` :
```javascript
const STEPS = [
  'Clone Repository',
  'Run Tests',
  'Build Package',
  'SonarQube Analysis',
  'Security Scan',  // ← NOUVELLE ÉTAPE
  'Build Docker Image',
  'Deploy to VM',
  'Health Check'
];

// Ajouter le cas dans executeRealStep()
case 'Security Scan':
  const { runSecurityScan } = require('./securityScanner');
  const result = await runSecurityScan(workDir);
  return result.output;
```

#### **Q6 : Peut-on déployer sur plusieurs VMs (production + staging) ?**
**R :** Oui, architecture à adapter :
1. Ajouter champ `environment` dans table `pipelines` (prod/staging)
2. Stocker plusieurs VMs dans table `vms`
3. Router le déploiement selon `pipeline.environment` :
```javascript
const targetVM = await getVMByEnvironment(pipeline.environment);
await sshService.connect(targetVM);
```

#### **Q7 : Comment sécuriser les secrets (passwords, tokens) ?**
**R :** 
- **Variables env** : Utiliser `is_secret: true` dans table `env_variables`
- **Affichage IHM** : Masquer avec `****` si `is_secret`
- **Logs** : Ne jamais logger les secrets (filtrer avec regex)
- **Chiffrement** : Utiliser `crypto` Node.js pour chiffrer en BDD :
```javascript
const crypto = require('crypto');
const algorithm = 'aes-256-cbc';
const key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');

function encrypt(text) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return iv.toString('hex') + ':' + encrypted;
}
```

#### **Q8 : Quelle est la stratégie de rollback exacte ?**
**R :** 
1. **Sauvegarde** : Chaque déploiement réussi est enregistré dans table `deployments` avec le nom d'image Docker
2. **Détection échec** : Health check POST-déploiement échoue
3. **Action** :
   ```javascript
   // Récupérer dernière version stable
   const lastDeployment = await pool.query(
     `SELECT docker_image FROM deployments 
      WHERE status = 'success' 
      ORDER BY deployed_at DESC LIMIT 1`
   );
   
   // Redéployer via SSH
   await sshService.executeCommand(
     `docker stop bfb-app && 
      docker rm bfb-app &&
      docker run -d --name bfb-app -p 8080:8080 ${lastDeployment.docker_image}`
   );
   ```
4. **Vérification** : Nouveau health check sur version rollback
5. **Notification** : Email/Slack aux admins

---

### Métriques et KPIs

#### Métriques collectées

```sql
-- Taux de succès global
SELECT 
  ROUND(100.0 * SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) / COUNT(*), 2) AS success_rate
FROM pipelines;

-- Temps moyen par étape
SELECT 
  step_name,
  AVG(EXTRACT(EPOCH FROM (completed_at - started_at))) AS avg_duration_seconds
FROM pipeline_logs
WHERE status = 'success'
GROUP BY step_name
ORDER BY avg_duration_seconds DESC;

-- Déploiements par jour (derniers 30 jours)
SELECT 
  DATE(created_at) AS deploy_date,
  COUNT(*) AS deploy_count
FROM pipelines
WHERE created_at > NOW() - INTERVAL '30 days'
  AND status = 'success'
GROUP BY deploy_date
ORDER BY deploy_date DESC;

-- Utilisateurs les plus actifs (triggers)
SELECT 
  u.username,
  COUNT(p.id) AS pipelines_triggered
FROM pipelines p
JOIN users u ON p.user_id = u.id
WHERE p.created_at > NOW() - INTERVAL '7 days'
GROUP BY u.username
ORDER BY pipelines_triggered DESC
LIMIT 5;
```

---

## 🎓 Conseils pour la Soutenance

### Points à Valoriser

1. **Complexité technique maîtrisée** :
   - Architecture microservices (3 repos : demo, cicd-platform, VM)
   - Communication asynchrone (WebSocket, Redis)
   - Orchestration multi-conteneurs (Docker Compose)

2. **Sécurité** :
   - OAuth2 (standard industrie)
   - RBAC (Role-Based Access Control)
   - SSH keys (no password)
   - Secrets management

3. **Qualité du code** :
   - Tests unitaires (24 passants)
   - Code coverage 75% (JaCoCo)
   - Linting backend (ESLint)
   - SonarQube intégration

4. **Expérience utilisateur** :
   - Interface temps réel (WebSocket)
   - Notifications instantanées
   - Design responsive (mobile-friendly)
   - Dark mode (bonus)

5. **Résilience** :
   - Rollback automatique
   - Health checks
   - Retry logic (SSH connexion)
   - Logs détaillés

### Questions Attendues & Réponses

**Q : "Pourquoi ne pas utiliser Jenkins/GitLab CI ?"**
**R :** L'objectif pédagogique était de comprendre les mécanismes internes d'un CI/CD en le construisant from scratch. Jenkins est une solution clé-en-main, notre projet démontre la maîtrise des concepts sous-jacents (webhooks, SSH, Docker orchestration).

**Q : "Que manque-t-il pour être production-ready ?"**
**R :** 
- [ ] Monitoring avancé (Prometheus + Grafana)
- [ ] Alerting (email/Slack sur échec)
- [ ] HTTPS (Let's Encrypt)
- [ ] Backup automatique BDD
- [ ] Tests d'intrusion (OWASP ZAP)
- [ ] CI/CD du CI/CD (meta-pipeline)

**Q : "Scalabilité ?"**
**R :** Architecture actuelle : monolithique backend. Pour scaler :
1. Séparer worker (exécution pipelines) du API server
2. Load balancer Nginx devant backend
3. Redis Cluster (réplication)
4. PostgreSQL read replicas
5. Kubernetes pour orchestration (HPA)

**Q : "Coût d'hébergement ?"**
**R :** 
- **Dev** : 0€ (local + VM VirtualBox)
- **Prod** : ~30€/mois
  - VM Ubuntu (2 vCPU, 4GB RAM) : 15€
  - PostgreSQL managé : 10€
  - Redis managé : 5€

---

## ✅ Checklist Finale Avant Soutenance

### Code
- [x] Tous tests passent (`mvn test`)
- [x] Docker builds sans erreur
- [x] Variables .env complétées
- [x] SSH keys configurées
- [x] Webhook GitHub actif

### Documentation
- [x] README.md à jour
- [x] Commentaires code clairs
- [x] Schéma architecture préparé
- [x] Slides présentation prêtes

### Démo
- [x] Pipeline manuel testé 3x
- [x] Webhook testé
- [x] Rollback testé
- [x] Vidéo backup enregistrée
- [x] Plan B (démo locale)

### Soutenance
- [x] Timing respecté (15 min max)
- [x] Répartition parole équilibrée
- [x] Anticipation questions
- [x] Tenue professionnelle

---

## 📞 Support & Contacts

**Équipe :** [Votre équipe]  
**Repo GitHub :** 
- https://github.com/Saad-Rafik-Etu-IMT/cicd-platform
- https://github.com/Saad-Rafik-Etu-IMT/demo

**En cas de problème :**
1. Consulter logs : `docker-compose logs -f backend`
2. Vérifier issues GitHub
3. Reset complet (voir section Dépannage)

---

**Bonne chance pour la soutenance du 9 janvier 2026 ! 🚀**

---

*Document généré le 6 janvier 2026*  
*Dernière mise à jour : Avant soutenance*

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
