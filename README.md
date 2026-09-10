# Securo Coolify

Ce dépôt permet de déployer [Securo](https://github.com/securo-finance/securo), une application open-source de gestion financière personnelle avec synchronisation bancaire automatique, sur [Coolify](https://coolifyhq.com) en utilisant le build pack Docker Compose.

Les images officielles sont tirées depuis `ghcr.io/securo-finance` avec une version épinglée par défaut (0.15.1).

---

## Déploiement sur Coolify

1. Dans votre projet Coolify, allez dans **New Resource**
2. Sélectionnez **Public Repository**
3. Entrez l'URL du dépôt : `https://github.com/welcoMattic/securo-coolify`
4. Choisissez la branche `main`
5. Sélectionnez le **Build Pack** : *Docker Compose*
6. Laissez **Docker Compose Location** à `/docker-compose.yaml` (emplacement par défaut)
7. Cliquez sur **Continue** puis **Save**
8. Dans le service `frontend`, définissez votre **domaine** (ex: `https://securo.coolify.kloude.fr`)
9. Dans l'onglet **Environment Variables** du service, vérifiez et configurez les variables nécessaires (voir section [Variables](#variables))
10. Cliquez sur **Deploy**

Le premier compte administrateur peut être créé directement via l'interface utilisateur (l'authentification locale est activée par défaut avec `LOCAL_AUTH_ENABLED=true`).

---

## Variables

Coolify génère automatiquement certaines variables :

| Variable | Description | Générée par Coolify |
|----------|-------------|-------------------|
| `SERVICE_PASSWORD_64_SECRETKEY` | Clé secrète du backend | ✅ Oui |
| `SERVICE_PASSWORD_POSTGRES` | Mot de passe PostgreSQL | ✅ Oui |
| `SERVICE_URL_FRONTEND` | URL publique du frontend | ✅ Oui |
| `SERVICE_FQDN_FRONTEND_8080` | Déclare le routage proxy vers le port 8080 du frontend | ✅ Oui |

Variables optionnelles à configurer manuellement selon vos besoins :

| Variable | Description | Valeur par défaut |
|----------|-------------|------------------|
| `SECURO_VERSION` | Version des images backend et frontend | `0.15.1` |
| `PLUGGY_CLIENT_ID` | Identifiant client Pluggy | - |
| `PLUGGY_CLIENT_SECRET` | Clé secrète Pluggy | - |
| `PLUGGY_OAUTH_REDIRECT_URI` | URL de callback OAuth Pluggy | - |
| `ENABLE_BANKING_APP_ID` | Identifiant application Enable Banking | - |
| `ENABLE_BANKING_PRIVATE_KEY` | Clé privée PEM (brut, avec `\n` échappés) | - |
| `ENABLE_BANKING_OAUTH_REDIRECT_URI` | URL de callback OAuth Enable Banking | - |
| `ENABLE_BANKING_API_URL` | URL de l'API Enable Banking | `https://api.enablebanking.com` |
| `SIMPLEFIN_ENABLED` | Activer SimpleFIN | `false` |
| `SIMPLEFIN_API_URL` | URL de l'API SimpleFIN | `https://beta-bridge.simplefin.org` |
| `OIDC_ENABLED` | Activer OpenID Connect | `false` |
| `OIDC_DISCOVERY_URL` | URL de discovery OIDC | - |
| `OIDC_CLIENT_ID` | Identifiant client OIDC | - |
| `OIDC_CLIENT_SECRET` | Clé secrète OIDC | - |
| `OIDC_REDIRECT_URI` | URL de callback OIDC | - |
| `OIDC_SCOPES` | Portées OIDC | `openid email profile` |
| `LOCAL_AUTH_ENABLED` | Activer l'inscription locale | `true` |
| `OPENEXCHANGERATES_APP_ID` | Identifiant Open Exchange Rates | - |
| `FX_SYNC_MODE` | Mode de synchronisation des taux | `on_demand` |
| `TESOURO_DIRETO_ENABLED` | Activer Tesouro Direto | `false` |
| `WEBAUTHN_RP_ID` | Identifiant Relying Party | - |
| `WEBAUTHN_RP_NAME` | Nom de l'application | `Securo` |
| `TRUSTED_PROXY_HOPS` | Sauts de proxy de confiance (coolify-proxy + nginx) | `2` |

> **Note sur les callbacks OAuth** :
> - Pour Pluggy et Enable Banking : `${FRONTEND_URL}/oauth/callback`
> - Pour OIDC : `${FRONTEND_URL}/api/auth/oidc/callback`

---

## Mise à jour

Pour mettre à jour Securo :

1. Dans Coolify, modifiez la variable `SECURO_VERSION` dans l'onglet **Environment Variables** du service (ex: `0.15.2`)
   - Ou modifiez directement la valeur par défaut dans le fichier `docker-compose.yaml`
2. Cliquez sur **Redeploy**

Les migrations de base de données (Alembic) sont exécutées automatiquement au démarrage du service `backend`.

Les releases upstream sont disponibles sur [la page releases](https://github.com/securo-finance/securo/releases).

---

## Agents IA / MCP

Le service `mcp-server` (pour les fonctionnalités d'agents IA) **n'est pas inclus** dans cette configuration. Il est disponible sous le profil `agents` dans le compose upstream.

Si vous souhaitez activer les agents IA, vous devez ajouter manuellement le service `mcp-server` et configurer les variables associées (`AGENTS_*`, voir `docker-compose.prod.yml` upstream).

---

## Volumes persistants

Les données suivantes sont stockées dans des volumes Docker persistants :

| Volume | Description |
|--------|-------------|
| `pgdata` | Données PostgreSQL (base de données principale) |
| `attachments` | Pièces jointes (factures, relevés bancaires, etc.) |
| `agent_knowledge` | Base de connaissances pour les agents IA |
| `agent_embedding_models` | Modèles d'embedding pour les agents IA |

Ces volumes sont conservés entre les redéploiements. Pour une sauvegarde complète, sauvegardez ces volumes avant une migration ou une suppression du service.
