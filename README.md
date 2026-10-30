# TaskF0RCE/.github

Le repo de configuration de l'orga. GitHub le traite à part : ce qu'il contient s'applique à **tous** nos repos.

> ⚠️ Ce repo doit rester **public** : c'est la condition pour que les templates par défaut et les workflows réutilisables marchent aussi dans les repos privés.

| Fichier | Rôle |
| --- | --- |
| `.github/ISSUE_TEMPLATE/` | Formulaires d'issue par défaut (bug, feature), pour les repos qui n'ont pas les leurs. |
| `.github/PULL_REQUEST_TEMPLATE.md` | Template de PR par défaut. |
| `.github/workflows/ci.yml` | **CI réutilisable** : détecte la stack (Node, Python, Maven, Gradle) et lance lint/tests/build. |
| `.github/workflows/release.yml` | **Release réutilisable** avec release-please : version SemVer, CHANGELOG, release GitHub. |
| `.github/workflows/pr-title.yml` | Vérifie que le titre de PR suit les Conventional Commits. |
| `labels.json` | La liste des labels standards. Le dashboard (Admin → Synchroniser les labels) la copie dans tous les repos. |
| `rulesets/protection-main.json` | Ruleset d'orga, prêt pour le jour où on passe au plan Team. |
| `profile/README.md` | La page d'accueil de l'orga sur github.com/TaskF0RCE. |

## Comment un projet utilise la CI et la release

Chaque projet créé depuis [`project-template`](https://github.com/TaskF0RCE/project-template) contient trois petits fichiers qui appellent ceux d'ici :

```yaml
jobs:
  ci:
    uses: TaskF0RCE/.github/.github/workflows/ci.yml@main
```

Corriger un workflow ici le corrige donc pour tous les projets d'un coup. Pour un projet dont le code n'est pas à la racine :

```yaml
    with:
      working-directory: app
```

## Kanban

Un seul tableau pour toutes les issues et PR de l'orga. À faire **une fois** :

1. Va sur https://github.com/orgs/TaskF0RCE/projects → **New project** → modèle **Board**.
2. Nomme-le exactement **`Kanban TaskF0RCE`** (le dashboard le cherche par ce nom).
3. Dans le tableau, clique sur **⋯** à côté du nom d'une colonne → **Edit details** du champ **Status**, puis :
   - renomme `Todo` en **`À faire`**,
   - renomme `In Progress` en **`En cours`**,
   - ajoute **`En review`** et place-la entre « En cours » et « Done »,
   - renomme `Done` en **`Fini`**.
4. Menu **⋯** en haut à droite → **Workflows** :
   - **Item closed** → Status = `Fini` → *Turn on*,
   - **Pull request merged** → Status = `Fini` → *Turn on*,
   - **Item reopened** → Status = `En cours` → *Turn on*,
   - **Item added to project** : laisse-le **désactivé**. Il écraserait la colonne choisie par le dashboard (« En cours »).
5. Regarde l'URL du projet : `…/projects/1`. Si le numéro n'est pas `1`, remplace `TaskF0RCE/1` dans `.github/ISSUE_TEMPLATE/*.yml` (ici et dans `project-template`).
6. Dans le dashboard : **Admin → Synchroniser le kanban** pour y ranger les issues et PR déjà ouvertes.

Ensuite, tout est automatique :

| Événement | Carte |
| --- | --- |
| Bouton « Nouvelle feature » du dashboard | issue + PR dans **En cours** |
| Issue créée via un formulaire | « No Status », puis **À faire** au prochain « Synchroniser le kanban » (ou glisse-la) |
| Bouton « Prête pour review » | PR dans **En review** |
| PR mergée / issue fermée (par « Closes #… ») | **Fini** |

En ligne de commande, l'étape 1 donne :

```bash
gh auth refresh -s project
```

```bash
gh project create --owner TaskF0RCE --title "Kanban TaskF0RCE"
```

## Protection de `main`

On veut partout : PR obligatoire, 1 approbation, CI verte, pas de push direct ni forcé.

- **Plan gratuit (actuel)** : GitHub n'accepte les rulesets d'orga qu'à partir du plan Team, et les rulesets de repo ne s'appliquent qu'aux repos **publics**. Le dashboard pose donc un ruleset sur chaque repo (Admin → Protéger main, et automatiquement à la création d'un projet). Pour les repos **privés**, deux garde-fous le remplacent :
  - le bouton « Merger » du dashboard reste grisé tant que la CI n'est pas verte et la PR pas approuvée,
  - le job `garde-fou main` de la CI passe au rouge si quelqu'un pousse directement sur `main`.
- **Si on passe au plan Team** : le ruleset de repo du dashboard s'applique aussi aux repos privés. On peut en plus créer le ruleset d'orga :

```bash
gh api --method POST /orgs/TaskF0RCE/rulesets --input rulesets/protection-main.json
```

## Réglages de l'orga à faire une fois

Dans https://github.com/organizations/TaskF0RCE/settings :

1. **Actions → General → Workflow permissions** : coche *Read repository contents and packages permissions* **et** *Allow GitHub Actions to create and approve pull requests* (sinon release-please ne peut pas ouvrir la PR de release).
2. **Personal access tokens → Settings** : autorise les *fine-grained personal access tokens* (pour le dashboard).
3. **Repository → Repository defaults** : nom de branche par défaut `main`.
