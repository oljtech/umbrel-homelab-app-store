# Homelab Apps — Umbrel Community App Store

A personal [Umbrel](https://umbrel.com) community app store.

## Apps

| App | Version | Description |
| --- | --- | --- |
| [Homepage](https://gethomepage.dev) | 1.13.2 | Modern, highly customizable application dashboard |
| [Picsou Finance](https://github.com/Zoeille/picsou-finance) | 1.0.14 | Self-hosted personal finance dashboard (banks, brokers, crypto) |

## Ajouter ce store à Umbrel

1. Push ce repo sur GitHub (voir plus bas).
2. Dans umbrelOS : **App Store** → menu **⋯** (en haut à droite) → **Community App Stores**.
3. Colle l'URL du repo, p. ex. `https://github.com/<ton-user>/umbrel-homelab-app-store`, puis **Add**.
4. Le store « Homelab Apps » apparaît ; installe **Homepage** depuis là.

> L'URL acceptée est celle du repo GitHub (https). Le repo doit être **public**.

## Structure (format imposé par Umbrel)

```
umbrel-homelab-app-store/
├── umbrel-app-store.yml        # id + nom du store
└── homelab-homepage/           # = <store-id>-<app>, préfixe obligatoire
    ├── umbrel-app.yml          # manifeste de l'app
    ├── docker-compose.yml      # service + app_proxy
    └── (1.jpg, 2.jpg, 3.jpg)   # captures pour la fiche store (à ajouter)
```

⚠️ **Renommer le store** : si tu changes `id:` dans `umbrel-app-store.yml`, tu dois
aussi renommer le dossier `homelab-homepage/`, le champ `id:` du manifeste **et**
`APP_HOST` dans le `docker-compose.yml` (toujours `<app-id>_web_1`).

## Notes Homepage spécifiques à Umbrel

- **`HOMEPAGE_ALLOWED_HOSTS`** : Homepage rejette tout Host header non listé. Réglé sur
  `*` par défaut ici (simple et fiable sur LAN). Pour verrouiller, mets
  `umbrel.local:8095,<IP>:8095` dans le `docker-compose.yml`.
- **Port** : exposé sur `8095` (champ `port` du manifeste). Change-le si conflit.
- **Docker socket** monté en lecture seule pour l'auto-découverte des conteneurs.
- La config vit dans `app/data/config` (dans le data dir de l'app sur Umbrel) et est
  éditable en YAML ; les fichiers par défaut sont créés au premier démarrage.

## ⚠️ Piège réseau Umbrel (toutes les apps)

Toutes les apps partagent le réseau Docker `umbrel_main_network` : les noms de
services (`db`, `web`…) sont des alias DNS **globaux** qui collisionnent entre
apps. Dans un compose Umbrel, toujours référencer les autres services par
**nom de conteneur** `<app-id>_<service>_1` (ex.
`homelab-picsou-finance_db_1`), comme le font les apps officielles.

## Notes Picsou Finance spécifiques à Umbrel

- **Port** : exposé sur `8577` (champ `port` du manifeste). Change-le si conflit.
- **3 conteneurs** : `server` (Spring Boot, port interne 8080), `tr-auth`
  (sidecar Trade Republic) et `db` (PostgreSQL 16). Le mot de passe Postgres est
  dérivé de `${APP_SEED}` (secret déterministe fourni par umbrelOS à chaque app).
- **Enable Banking** (synchro bancaire PSD2) : déposer la clé
  `enablebanking.pem` dans `data/secrets/` du data dir de l'app (montée en
  lecture seule sur `/run/secrets/`). Optionnel — sans elle, le reste fonctionne.
- **Données** : app dans `data/app/` (secrets auto-générés, clé du wizard),
  base dans `data/postgres/`. À inclure dans les sauvegardes.
- **Postgres 16 épinglé** : Renovate ne propose que les digests, pas les
  majeures (une MAJ 16→17 exigerait une migration manuelle du datadir).

## Mettre à jour Homepage

Bump `version:` dans le manifeste + l'image (`tag@sha256:...`) dans le compose,
commit/push, puis « Update » dans Umbrel. Récupérer le digest d'un tag :

```sh
token=$(curl -s "https://ghcr.io/token?scope=repository:gethomepage/homepage:pull" \
  | sed -E 's/.*"token":"([^"]+)".*/\1/')
curl -sI -H "Authorization: Bearer $token" \
  -H "Accept: application/vnd.oci.image.index.v1+json" \
  "https://ghcr.io/v2/gethomepage/homepage/manifests/<TAG>" \
  | grep -i docker-content-digest
```
