# Authentification Server — Rapport technique

## 1. Présentation

Auth·entification Server est une API REST Java (Spring Boot) fournissant un service d'authentification simple basé sur JWT (JSON Web Token). Le projet utilise Spring Security, Spring Data JPA et une base de données en mémoire H2 pour le stockage des utilisateurs.

Objectif : proposer un serveur d'authentification minimal mais complet pour des démonstrations, prototypes ou comme point de départ pour une solution plus complète.

---

## 2. Sommaire

- Présentation
- Architecture & composants
- Modèle de données
- Endpoints disponibles (contrôleurs, payloads, exemples)
- Sécurité (JWT)
- Configuration (application.properties)
- Exécution locale, build & tests

---

## 3. Architecture & composants

Composants principaux :

- Spring Boot (application principale)
- Spring Security (gestion de l'authentification)
- Spring Data JPA + H2 (persistance en mémoire pour le développement)
- jjwt (génération et validation des JWT)
- BCrypt (hash des mots de passe)

Organisation du code (paquetage racine : `org.example.authentificationserver`):

- `controller` : point d'entrée HTTP (AuthenticationController)
- `service` : logique métier (AuthenticationService, JwtService)
- `config` : configuration Spring/Security (SecurityConfiguration, JwtAuthenticationFilter, ApplicationConfiguration)
- `repository` : couche d'accès aux données (UserRepository)
- `entity` : entités JPA (User)
- `dto` : objets de transfert (RegisterUserDto, LoginUserDto, LoginResponse)

---

## 4. Modèle de données

Entité principale : `User` (table `users`)

Champs importants :

- `id`: Integer — identifiant automatique
- `fullName`: String — nom complet de l'utilisateur
- `email`: String (unique) — identifiant/login
- `password`: String — mot de passe hashé (BCrypt)
- `createdAt`, `updatedAt`: timestamps gérés automatiquement

L'entité implémente `UserDetails` pour l'intégration avec Spring Security.

---

## 5. Endpoints disponibles

Base path : `/auth`

1) POST /auth/signup
- Description : enregistre un nouvel utilisateur
- Request body (JSON) :
  {
    "email": "user@example.com",
    "password": "P@ssw0rd",
    "fullName": "Prénom Nom"
  }
- Response (200 OK) : l'objet `User` enregistré (sensible aux champs exposés — ici l'entité entière est retournée)

Exemple curl :

```bash
curl -X POST http://localhost:8081/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret","fullName":"Test User"}'
```

2) POST /auth/login
- Description : authentifie un utilisateur et retourne un JWT
- Request body (JSON) :
  {
    "email": "user@example.com",
    "password": "P@ssw0rd"
  }
- Response (200 OK) :
  {
    "token": "<JWT_TOKEN>",
    "expiresIn": 3600000
  }

Exemple curl :

```bash
curl -X POST http://localhost:8081/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"secret"}'
```

Utilisation du JWT pour accéder à une route protégée (exemple générique) :

```bash
curl -H "Authorization: Bearer <JWT_TOKEN>" http://localhost:8081/protected/resource
```

---

## 6. Sécurité & JWT

- Génération : `JwtService` se charge de créer le token (algorithme HS256) avec :
  - subject = email
  - issuedAt = maintenant
  - expiration = maintenant + `security.jwt.expiration-time`
- Validation : `JwtAuthenticationFilter` extrait le header `Authorization` (`Bearer <token>`), vérifie le token et place l'Authentication dans le `SecurityContext`.

Paramètres (fichier de configuration) :
- `security.jwt.secret-key` : clé secrète encodée en base64 (utilisée pour signer/verifier les JWT). Actuellement la valeur par défaut est dans `application.properties` pour développement. En production, fournissez-la via une variable d'environnement ou un gestionnaire de secrets.
- `security.jwt.expiration-time` : durée d'expiration en millisecondes (ex : 3600000 = 1 heure)

Conseils de sécurité :
- Ne committez jamais de clés secrètes en clair dans le dépôt.
- Utilisez une clé suffisamment longue et stockée dans un coffre-fort (Vault, AWS Secrets Manager, etc.).
- Pensez au rafraîchissement/revocation des tokens pour une solution complète.

---

## 7. Configuration (application.properties)

Fichier principal : `src/main/resources/application.properties`

Valeurs par défaut utiles :

- spring.application.name=authentification-server
- server.port=8081
- spring.h2.console.enabled=true
- spring.datasource.url=jdbc:h2:mem:authdb
- security.jwt.secret-key=<clé_base64>
- security.jwt.expiration-time=3600000

Console H2 (pour développement) : http://localhost:8081/h2-console
JDBC URL (par défaut) : jdbc:h2:mem:authdb

---

## 8. Exécution locale — Prérequis & étapes

Prérequis :
- JDK 17
- Maven (le projet contient les wrappers `mvnw`/`mvnw.cmd`)

Build & exécution (depuis la racine du projet) :

Sur Windows (cmd.exe) :

```cmd
mvnw.cmd -DskipTests package
mvnw.cmd -DskipTests spring-boot:run
```

Ou directement avec Maven installé :

```cmd
mvn -DskipTests package
mvn -DskipTests spring-boot:run
```

L'application écoute par défaut sur le port 8081. Testez les endpoints décrits précédemment avec curl ou Postman.

---

