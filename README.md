# podcast (public) — coquille de déploiement uniquement

Ce repo public **ne contient aucun fichier audio**. Il ne sert qu'à publier
le site sur GitHub Pages.

## Architecture

- `fox3000foxy/podcast-src` (privé) : les 953 `.m4a` + `index.html` + `track_numbers.json`.
  Son workflow `trigger-public-deploy.yml` envoie un `repository_dispatch`
  à chaque push des sources.
- Ce repo (public) : le workflow `deploy-from-private-src.yml` checkout
  les sources **dans le workspace éphémère du runner** via
  `SRC_READ_TOKEN`, assemble `site/` (`index.html` + `files/*.m4a`),
  puis déploie sur Pages avec `actions/deploy-pages`.

Les `.m4a` ne sont présents ni dans l'historique git de ce repo,
ni dans ses PRs : uniquement servis par le site Pages généré.
(`retention-days: 1` limite aussi la durée de vie de l'artifact intermédiaire.)

## Garde-fou

`.gitignore` bloque tout `*.m4a` : impossible de commiter un audio ici par erreur.

## Secrets requis

| Secret | Repo | Contenu |
|---|---|---|
| `DISPATCH_TOKEN` | `podcast-src` | PAT avec `Contents: read & write` sur `podcast` (déclenche le dispatch) |
| `SRC_READ_TOKEN` | `podcast` (ce repo) | PAT avec `Contents: read` sur `podcast-src` (checkout des sources) |

Un seul PAT classique avec scope `repo` couvre les deux usages.

## Limite connue

Le payload fait ~2,6 Go ; GitHub Pages annonce un site publié ≤ 1 Go
et un timeout de déploiement de 10 min. Le déploiement peut donc échouer —
voir le run d'Actions pour l'erreur exacte si c'est le cas.
