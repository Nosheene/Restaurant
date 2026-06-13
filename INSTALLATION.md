# Restaurant — Fil rouge Backend Studi

Projet fil rouge de la formation **Développeur Backend Studi** : API REST Symfony pour la gestion d'un restaurant.

**Emplacement :** `/Volumes/LaCie/DP Front et back/Restaurant`

**Dépôt GitHub :** https://github.com/Nosheene/Restaurant

## Prérequis

- PHP >= 8.2
- Composer
- Symfony CLI
- MySQL 8 (recommandé par Studi) ou SQLite (configuré par défaut en local)

## Installation

```bash
cd "/Volumes/LaCie/DP Front et back/Restaurant"
composer install
cp .env .env.local   # si .env.local n'existe pas encore
```

### Base de données SQLite (développement local)

Le fichier `.env.local` est déjà configuré avec SQLite :

```
DATABASE_URL="sqlite:///%kernel.project_dir%/var/data.db"
```

```bash
php bin/console doctrine:schema:create
php bin/console doctrine:fixtures:load --no-interaction
```

### Base de données MySQL (conforme au cours Studi)

Dans `.env.local`, remplacez la ligne `DATABASE_URL` par :

```
DATABASE_URL="mysql://root:VOTRE_MOT_DE_PASSE@127.0.0.1:3306/restaurant?serverVersion=8&charset=utf8mb4"
```

Puis :

```bash
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate
php bin/console doctrine:fixtures:load --no-interaction
```

## Lancer le serveur

```bash
symfony server:start
symfony server:status   # noter le port affiché (8000, 8002…)
```

L'API est disponible sur `http://127.0.0.1:8000` (ou le port indiqué par `symfony server:status`).

## Documentation API (Swagger)

- Interface : http://127.0.0.1:8000/api/doc
- JSON : http://127.0.0.1:8000/api/doc.json

## Endpoints principaux

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/api/registration` | Inscription |
| POST | `/api/login` | Connexion (retourne un `apiToken`) |
| GET | `/api/account/me` | Profil utilisateur (authentifié) |
| PUT | `/api/account/edit` | Modifier son compte |
| POST | `/api/restaurant` | Créer un restaurant |
| GET | `/api/restaurant/{id}` | Lire un restaurant |
| PUT | `/api/restaurant/{id}` | Modifier un restaurant |
| DELETE | `/api/restaurant/{id}` | Supprimer un restaurant |

## Authentification

Les routes protégées nécessitent le header :

```
X-AUTH-TOKEN: <votre_token>
```

### Comptes de test (fixtures)

- Email : `email.1@studi.fr` → Mot de passe : `password1`
- Email : `email.2@studi.fr` → Mot de passe : `password2`
- etc. (20 utilisateurs générés)

Exemple de connexion :

```bash
curl -X POST http://127.0.0.1:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"email.1@studi.fr","password":"password1"}'
```

## Structure du projet

```
restaurant/
├── bin/              # Console Symfony
├── config/           # Configuration (sécurité, Doctrine, routes…)
├── migrations/       # Migrations Doctrine (MySQL)
├── public/           # Point d'entrée web
├── src/
│   ├── Controller/   # Contrôleurs API
│   ├── DataFixtures/ # Données de test
│   ├── Entity/       # Entités (Restaurant, User, Picture)
│   ├── Repository/   # Repositories Doctrine
│   ├── Security/     # Authentification par token
│   └── Service/      # Services métier
├── templates/
├── tests/
└── var/              # Cache, logs, base SQLite
```

## Tests Postman

Collection exportable : `postman/Restaurant-API.postman_collection.json`

1. Importer la collection dans Postman.
2. Vérifier la variable `base_url` (`http://127.0.0.1:8000` ou le port affiché par `symfony server:status` en local).
3. Exécuter **Login** pour alimenter `api_token` automatiquement.
4. Lancer le **Collection Runner** pour valider tous les endpoints.

Guide captures DP : `Guide captures ecran — Tests Postman Restaurant.md` (dossier DP Front et back).

## Déploiement Upsun (Platform.sh)

**Plateforme :** [Upsun](https://upsun.com) (ex-Platform.sh) — format **Flex**  
**Fichier de configuration :** `.upsun/config.yaml` (PHP 8.2, MySQL 10.6, hooks Symfony)  
**Projet cloud :** Restaurant (`vcl763wv52oem`)  
**URL de production :** https://main-bvxea6i-vcl763wv52oem.fr-3.platformsh.site

### Prérequis

1. Compte Upsun / Platform.sh : https://console.platform.sh/signup  
2. Symfony CLI : `symfony version`  
3. Git configuré dans le projet

### Connexion

```bash
cd "/Volumes/LaCie/DP Front et back/Restaurant"
symfony login
symfony cloud:auth:info
```

### Créer ou lier le projet cloud

```bash
symfony cloud:project:create --title="Restaurant API Studi"
# ou, si le projet existe déjà :
symfony cloud:project:set-remote vcl763wv52oem
```

### Déployer

```bash
git add .
git commit -m "Mise à jour configuration Upsun"
symfony cloud:environment:push -e main --wait
# ou forcer un redeploy :
symfony cloud:environment:redeploy -e main --wait
```

Les hooks exécutés automatiquement :
- **build** : `symfony-build` (Composer, cache)
- **deploy** : `symfony-deploy` (migrations Doctrine)

### Vérifier le déploiement

```bash
symfony cloud:url -e main
```

Swagger en production :

```
https://main-bvxea6i-vcl763wv52oem.fr-3.platformsh.site/api/doc
```

Test login en production (inscription requise si les fixtures ne sont pas chargées) :

```bash
curl -X POST https://main-bvxea6i-vcl763wv52oem.fr-3.platformsh.site/api/registration \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Marie","lastName":"Martin","email":"marie.prod@studi.fr","password":"password1"}'

curl -X POST https://main-bvxea6i-vcl763wv52oem.fr-3.platformsh.site/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"marie.prod@studi.fr","password":"password1"}'
```

Dans Postman, définir `base_url` sur l'URL HTTPS de production.

### Comptes en production

Les fixtures (`email.1@studi.fr` / `password1`) sont chargées **en local** uniquement.  
En production, créer un compte via `POST /api/registration` ou charger les fixtures :

```bash
symfony cloud:ssh -e main
php bin/console doctrine:fixtures:load --no-interaction
exit
```

Procédure complète : `/Volumes/LaCie/IA Back DP/deploiement/README.md`

## Commandes utiles

```bash
php bin/console debug:router      # Lister les routes
php bin/console doctrine:schema:validate
php bin/phpunit                   # Lancer les tests
symfony check:requirements        # Vérifier les prérequis PHP
```
