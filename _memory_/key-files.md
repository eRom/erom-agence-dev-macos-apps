# Fichiers clés

Mise à jour : 2026-09-12

## Contrat et vitrine

- `CLAUDE.md` : contrat du dépôt, 7 invariants (dont pas de `commands/`, jargon en anglais), section Origine, État actuel.
- `README.md` : vitrine GitHub, carte en tête, tableau des 14 skills, section Origine.
- `assets/erom-dev-macos-apps.png` : carte au fusain, écorché d'une fenêtre d'app sur un portable.

## Plugin distribué

- `plugin/.claude-plugin/plugin.json` : manifeste 0.1.0, description longue alignée sur l'entrée marketplace, keywords choisis par Romain (sans `pwa`).
- `plugin/README.md` : même contenu que le README racine, sans image.
- `plugin/LICENSE` : MIT Romain Ecarnot + mention des parties dérivées d'OpenAI.
- `plugin/skills/build-run-debug/references/build-and-run-script.md` : source canonique des deux scripts bash (CLI SwiftPM, app GUI SwiftPM en bundle `.app`). Ex `run-button-bootstrap.md`, renommé.
- `plugin/skills/swiftui-patterns/references/components-index.md` : index des références UI.

## Hors dépôt, liés

- `~/dev/erom-marketplace/.claude-plugin/marketplace.json` : entrée du plugin, metadata 0.28.0.
- `~/dev/erom-agence-dev-plugin/plugin/skills/illustrate/references/GABARIT.md` : table des cartes livrées, ligne ajoutée le 2026-09-12.
- `.claude/settings.local.json` : profil `plugin` de session-profile (jamais commité).
