# Guide de déploiement — PEDAGO ERP sur Hostinger (Node.js Web App)

Ce guide est spécifique à l'hébergement mutualisé **Hostinger Business Web Hosting**
avec la fonctionnalité **Node.js Web App** de hPanel.

- Pas de VPS, pas d'accès root, pas de Nginx, pas de PM2, pas de Docker.
- Frontend + API sous le même domaine (ex. `https://ispm.pedago.ma`).
- Base SQLite installée dans un répertoire de données **persistant**.
- Installation totalement vierge : aucune donnée de démonstration.

---

## 1. Vue d'ensemble de l'application

| Élément | Valeur |
|---|---|
| Structure | Frontend React (pré-construit dans `server/public/`) + API Express (`server/dist/`) |
| Base de données | SQLite (fichier), créée automatiquement au premier démarrage |
| Point d'entrée | `server/app.cjs` |
| Migrations | Appliquées automatiquement à chaque démarrage (`app.cjs`) |
| Compte admin | Créé automatiquement si la base est vierge (via variables d'environnement) |

Le serveur Express sert **tout** : le site (frontend), l'API (`/api/...`) et les
documents (`/uploads/...`). Aucun serveur web supplémentaire n'est nécessaire.

---

## 2. Prérequis dans hPanel

1. **Le sous-domaine** : dans hPanel → *Domains* → `pedago.ma` (s'il est géré
   par Hostinger) → *Subdomains* → créez `ispm.pedago.ma`.
   Si le domaine est géré chez un autre registrar, créez d'abord le
   sous-domaine chez ce registrar avec un enregistrement `A` pointant vers
   l'IP de votre hébergement Hostinger (affichée dans hPanel → *Hosting*).
2. **Supprimer tout site existant** sur ce sous-domaine : la création d'une
   Node.js Web App exige un emplacement de domaine libre.

---

## 3. Préparer et déployer l'archive

1. Récupérez `pedago-ispm-hostinger.zip`.
2. hPanel → *Websites* → *Add Website* → **Node.js web app**.
3. Choisissez **Upload your files** et téléversez l'archive `.zip`.

---

## 4. Paramètres de déploiement (hPanel)

| Paramètre | Valeur à saisir |
|---|---|
| Framework preset | **Express** |
| Root directory | **`./`** (racine de l'archive) |
| Node.js version | **20** ou **22** |
| Build command | **`cd server && npm install --no-audit --no-fund && node node_modules/prisma/build/index.js generate`** |
| Output directory | *(vide)* |
| Package manager | npm |
| Entry file | **`server/app.cjs`** |

> **Important** : le runtime Hostinger n'a pas `npm` dans le PATH — tout doit
> être installé en phase de **build**. Le Build Command ci-dessus installe les
> dépendances du serveur et génère le client Prisma (avec les binaires Linux).
> Au démarrage, `app.cjs` ne lance aucun `npm`/`npx` : il vérifie les
> dépendances (message explicite si le build a été oublié), applique les
> migrations via le CLI Prisma embarqué, puis démarre Express.

---

## 5. Variables d'environnement (hPanel)

Dans les paramètres de déploiement → *Environment variables*, ajoutez :

| Variable | Valeur | Obligatoire |
|---|---|---|
| `JWT_SECRET` | longue valeur aléatoire (64+ caractères) | **OUI** |
| `DATA_DIR` | chemin de données persistant (voir ci-dessous) | **OUI** (recommandé) |
| `ADMIN_EMAIL` | email admin du client | Recommandé |
| `ADMIN_PASSWORD` | mot de passe admin du client | Recommandé |
| `ETABLISSEMENT` | nom affiché dans l'application | Recommandé |
| `PORT` | **NE PAS DÉFINIR** — fourni par la plateforme | — |
| `DATABASE_URL` | **NE PAS DÉFINIR** — géré par `app.cjs` | — |

### Trouver votre chemin `DATA_DIR`

Le répertoire de données doit être **en dehors du dossier de l'application**
(celui-ci est reconstruit à chaque déploiement). Ouvrez le *File Manager*
hPanel : le chemin affiché en racine est votre home, par exemple
`/home/u123456789`. Définissez alors :

```
DATA_DIR=/home/u123456789/domains/ispm.pedago.ma/erp-data
```

> Alternativement, si vous ne définissez pas `DATA_DIR`, l'application utilise
> par défaut `/<home>/pedago-data`. Attention : ce dossier serait partagé par
> toutes les applications Node.js du même compte — définissez `DATA_DIR` si
> vous hébergez plusieurs clients sur le même plan.

À chaque démarrage, l'application affiche dans les *Runtime Logs* le chemin
de données utilisé (utile pour vérifier).

### Générer le `JWT_SECRET`

Sur votre PC (PowerShell) :

```powershell
-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 64 | % {[char]$_})
```

ou sur n'importe quelle machine Linux : `openssl rand -base64 48`.

> Sans `JWT_SECRET`, les sessions de tous les utilisateurs sont invalidées à
> chaque redémarrage de l'application.

---

## 6. Premier démarrage

Cliquez **Deploy**. Le premier démarrage effectue automatiquement :

1. la création de `DATA_DIR` (base `erp.db` + dossiers `uploads/` et `uploads/branding/`) ;
2. les migrations Prisma (7 tables de base) ;
3. la création du compte administrateur et de l'année scolaire en cours.

Vérification : hPanel → votre site → **Runtime Logs**, vous devez voir :

```
[PEDAGO] Repertoire de donnees : /home/.../erp-data
[PEDAGO] Base de donnees       : /home/.../erp-data/erp.db
[PEDAGO] Dependances serveur presentes.
[PEDAGO] Client Prisma deja genere.
PEDAGO ERP Server running on port XXXX (etat : starting)
[PEDAGO] Migration 20260727151516_init...
[PEDAGO] Migration 20260727151516_init appliquee.
...
[PEDAGO] Schema de base a jour (7 migrations connues).
[PEDAGO] Initialisation terminee : etat ready (routes metier disponibles).
```

Le badge de statut du site doit passer à **Running**, et `/api/health`
doit renvoyer `"state":"ready"`.

---

## 7. Première connexion

1. Ouvrez `https://ispm.pedago.ma` — la page de connexion PEDAGO ERP s'affiche.
2. Connectez-vous avec `ADMIN_EMAIL` / `ADMIN_PASSWORD`.
3. **Changez immédiatement le mot de passe** via la page *Utilisateurs*.
4. Créez les autres comptes (Direction, Responsable pédagogique,
   Enseignant, Scolarité) depuis la même page.

L'installation est **vierge** : le client crée lui-même ses filières,
programmes, étudiants et paiements.

---

## 8. Ce qui est persistant (et ce qui ne l'est pas)

| Données | Emplacement | Persistant ? |
|---|---|---|
| Base SQLite `erp.db` | `DATA_DIR` | OUI |
| Documents étudiants | `DATA_DIR/uploads/` | OUI |
| Logo établissement (Réglages) | `DATA_DIR/uploads/branding/` | OUI |
| Code de l'application | dossier géré par Hostinger | Remplacé à chaque déploiement |

**Règle d'or** : ne stockez jamais de données dans le dossier de
l'application. Toutes les données vivent dans `DATA_DIR`.

---

## 9. Mises à jour de l'application

Pour appliquer une nouvelle version : téléversez la nouvelle archive
(*Deployments* → upload). Au redémarrage, les éventuelles nouvelles
migrations sont appliquées automatiquement et **les données du client sont
conservées** (elles ne sont pas dans l'archive).

---

## 10. Sauvegardes

Sauvegardez régulièrement via le *File Manager* hPanel (ou SSH) :

```
DATA_DIR/erp.db              ← base complète (le seul fichier essentiel)
DATA_DIR/uploads/            ← documents étudiants et logo
```

Exemple en SSH (Business inclut l'accès SSH) :

```bash
cp ~/domains/ispm.pedago.ma/erp-data/erp.db ~/backups/erp-$(date +%F).db
```

Restauration : remettre le fichier `erp.db` et/ou le dossier `uploads/`
en place, puis redémarrer l'application (*Running badge* → *Restart*).

---

## 11. Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `code 127` au démarrage (npm install) | `npm` absent du runtime | Utiliser le Build Command de la section 4 (tout s'installe au build, rien au runtime) |
| `Dependances serveur absentes` dans les logs | Build Command vide ou échoué | Vérifier le Build Command dans hPanel et relancer le déploiement |
| `schema-engine ... EAGAIN` au démarrage | Fork d'un binaire bloqué par l'hébergeur | Normal avec les anciennes versions ; v7+ applique les migrations **en processus** (aucun binaire spawné) — vérifier que `/api/health` renvoie `ready` |
| `FOREIGN KEY constraint failed` ou `UNIQUE constraint failed: new_*.id` pendant les migrations | Migrateur v7 : migrations de reconstruction de table exécutées sans désactiver les clés étrangères, et sans atomicité | Corrigé en v8 : chaque migration est atomique (transaction) avec FK désactivées, et reprend automatiquement d'un état partiel (tables `new_*` orphelines supprimées). Redéployer la v8 : aucune action manuelle requise, les données existantes sont conservées |
| `/api/...` renvoie 503 | Initialisation pas terminée ou échouée | Vérifier `/api/health` : `state:ready` attendu ; si `failed`, lire la dernière cause dans les Runtime Logs |
| Build green mais site ne répond pas | Variable d'environnement manquante | Vérifier `JWT_SECRET` et `DATA_DIR` dans les Runtime Logs |
| Sessions perdues à chaque redémarrage | `JWT_SECRET` non défini | Définir la variable puis *Restart* |
| `ispm.pedago.ma` ne répond pas | DNS non propagé | Vérifier l'enregistrement du sous-domaine, attendre la propagation |
| 403 après un déploiement | Cache `.htaccess` | Redéployer (Hostinger le régénère) |
| Page de connexion inaccessible / 404 API | Entry file incorrect | Vérifier `app.cjs` et Root directory `server` |
| Logo ne s'affiche plus | `DATA_DIR` modifié après coup | Remettre la même valeur qu'au premier déploiement |
| Erreur "SQLITE_BUSY" | Concurrence anormale d'écritures | Garder `connection_limit=1` (défaut via `app.cjs`) ; limiter les écritures simultanées massives |

---

## 12. Limites connues de SQLite sur hébergement mutualisé

SQLite est retenu ici car chaque client dispose de sa propre instance à
faible concurrence (un établissement scolaire). C'est fiable, sans
configuration et sauvegardable par simple copie de fichier.

En revanche, si un jour vous hébergez **plusieurs dizaines d'utilisateurs
simultanés avec de fortes écritures simultanées** sur une même instance, la
solution propre serait MySQL (inclus dans votre plan Hostinger). Cela
nécessiterait une adaptation du projet (migrations régénérées) — demandez-la
le moment venu, ce n'est pas inclus dans cette archive.
