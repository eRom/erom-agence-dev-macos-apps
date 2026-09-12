---
name: fix-codesign-error
description: "Inspecte un échec de signature ou d'entitlement macOS et explique le chemin de correction minimal. Commande explicite, lancée au slash : /erom-dev-macos-apps:fix-codesign-error."
argument-hint: "[app=...] [identity=...] [mode=inspect|repair-plan]"
user-invocable: true
disable-model-invocation: true
---

# /erom-dev-macos-apps:fix-codesign-error

Inspecte un échec de signature ou d'entitlement macOS et explique le chemin
de correction minimal.

Le détail de la procédure vit dans la skill `erom-dev-macos-apps:signing-entitlements` : la charger avec l'outil Skill avant de commencer.

## Arguments

- `app` : chemin vers le bundle `.app` ou le binaire (optionnel)
- `identity` : indice d'identité de signature (optionnel)
- `mode` : `inspect` ou `repair-plan` (optionnel, défaut : `inspect`)

Arguments reçus : $ARGUMENTS

Tous ces arguments sont optionnels. En leur absence, détecter l'app ou le
binaire concerné depuis le repo courant.

## Procédure

1. Inspecter le bundle de l'app, l'exécutable, les infos de signature, et les entitlements.
2. Déterminer si le problème vient de l'identité, du provisioning, du hardened runtime, du sandboxing, ou d'une trust policy.
3. Résumer la classe d'échec exacte en langage clair.
4. Fournir la séquence de réparation minimale ou la commande de validation.

## Garde-fous

- Ne jamais inventer un entitlement : le lire depuis le binaire ou les fichiers source.
- Distinguer les problèmes de signature de développement local des échecs de distribution ou de notarization.
- Préférer des commandes vérifiables comme `codesign -d`, `spctl`, et `plutil` plutôt que deviner.
