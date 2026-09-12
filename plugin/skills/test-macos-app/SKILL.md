---
name: test-macos-app
description: "Lance le plus petit scope de test macOS pertinent et explique les échecs par catégorie. Commande explicite, lancée au slash : /erom-dev-macos-apps:test-macos-app."
argument-hint: "[scheme=...] [target=...] [filter=...] [configuration=Debug|Release]"
user-invocable: true
disable-model-invocation: true
---

# /erom-dev-macos-apps:test-macos-app

Lance le plus petit scope de test macOS pertinent en premier et explique
les échecs par catégorie.

Le détail de la procédure vit dans la skill `erom-dev-macos-apps:test-triage` : la charger avec l'outil Skill avant de commencer.

## Arguments

- `scheme` : nom du scheme Xcode (optionnel)
- `target` : target ou nom de product de test (optionnel)
- `filter` : expression de filtre de test (optionnel)
- `configuration` : `Debug` ou `Release` (optionnel, défaut : `Debug`)

Arguments reçus : $ARGUMENTS

Tous ces arguments sont optionnels. En leur absence, détecter le harness de
test et le scope depuis le repo courant.

## Procédure

1. Détecter si le repo utilise `xcodebuild test` ou `swift test`.
2. Préférer une exécution de test ciblée quand un target ou un filter est fourni.
3. Classer les échecs comme compile, assertion, crash, env/setup, ou flake.
4. Résumer le blocage principal et la plus petite étape suivante sensée.

## Garde-fous

- Éviter de relancer la suite complète si un rerun ciblé est possible.
- Distinguer les échecs de build des vrais tests en échec.
- Signaler quand un setup de host app ou des suppositions de test simulator-only s'infiltrent dans un run macOS.
