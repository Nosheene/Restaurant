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
```

L'API est disponible sur `http://127.0.0.1:8000`.

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
2. Vérifier la variable `base_url` (`http://127.0.0.1:8000` en local).
3. Exécuter **Login** pour alimenter `api_token` automatiquement.
4. Lancer le **Collection Runner** pour valider tous les endpoints.

Guide captures DP : `Guide captures ecran — Tests Postman Restaurant.md` (dossier DP Front et back).

## Déploiement Platform.sh

Fichiers de configuration : `.platform.app.yaml`, `.platform/services.yaml`, `.platform/routes.yaml`

```bash
symfony login
symfony cloud:project:create --title="Restaurant API Studi"
symfony cloud:environment:push
symfony cloud:url
```

Procédure complète : `/Volumes/LaCie/IA Back DP/deploiement/README.md`

## Commandes utiles

```bash
php bin/console debug:router      # Lister les routes
php bin/console doctrine:schema:validate
php bin/phpunit                   # Lancer les tests
symfony check:requirements        # Vérifier les prérequis PHP
```
