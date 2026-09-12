---
name: build-and-run-macos-app
description: "Crée ou met à jour le script build_and_run.sh local au projet, puis l'utilise comme point d'entrée de build et run par défaut. Commande explicite, lancée au slash : /erom-dev-macos-apps:build-and-run-macos-app."
argument-hint: "[scheme=...] [workspace=...] [project=...] [product=...] [mode=run|debug|logs|telemetry|verify] [app_name=...]"
user-invocable: true
disable-model-invocation: true
---

# /erom-dev-macos-apps:build-and-run-macos-app

Crée ou met à jour le script `build_and_run.sh` local au projet, puis
utilise ce script comme point d'entrée de build/run par défaut.

Le détail de la procédure vit dans la skill `erom-dev-macos-apps:build-run-debug` : la charger avec l'outil Skill avant de commencer.

## Arguments

- `scheme` : nom du scheme Xcode (optionnel)
- `workspace` : chemin vers le `.xcworkspace` (optionnel)
- `project` : chemin vers le `.xcodeproj` (optionnel)
- `product` : nom de l'executable product SwiftPM (optionnel)
- `mode` : `run`, `debug`, `logs`, `telemetry`, ou `verify` (optionnel, défaut : `run`)
- `app_name` : nom du process/app à arrêter avant de relancer (optionnel)

Arguments reçus : $ARGUMENTS

Tous ces arguments sont optionnels. En leur absence, détecter la forme du
projet et la cible depuis le repo courant.

## Procédure

1. Détecter si le repo utilise un workspace Xcode, un project Xcode, ou un package SwiftPM.
2. Créer ou mettre à jour `script/build_and_run.sh` pour qu'il arrête toujours l'app en cours, builde la cible macOS, et lance le résultat frais.
3. Pour SwiftPM, garder le lancement direct de l'exécutable réservé aux vrais outils CLI ; pour les apps AppKit/SwiftUI GUI, créer un bundle `.app` local au projet et le lancer avec `/usr/bin/open -n`.
4. Supporter les flags optionnels de script `--debug`, `--logs`, `--telemetry`, et `--verify`.
5. Suivre le contrat de bootstrap canonique dans `${CLAUDE_PLUGIN_ROOT}/skills/build-run-debug/references/build-and-run-script.md` pour la forme exacte du script.
6. Lancer le script dans le mode demandé et résumer tout échec de build, de script, ou de lancement.

## Garde-fous

- Garder le chemin de script sans flag simple : kill, build, run.
- Utiliser `--debug`, `--logs`, `--telemetry`, ou `--verify` seulement quand l'utilisateur demande ces modes.
