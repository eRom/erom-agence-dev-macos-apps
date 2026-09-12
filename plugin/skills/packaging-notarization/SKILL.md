---
name: packaging-notarization
description: Prépare les workflows de packaging et de notarization macOS. À utiliser pour archiver une app, valider un bundle, ou expliquer un échec propre à la distribution.
---

# Packaging et notarization

## Démarrage rapide

Utiliser cette skill quand le travail porte sur l'expédition de l'app plutôt
que sur son simple run local : archives, app bundles exportés, préparation à
la notarization, hardened runtime, ou validation de distribution.

## Procédure

1. Confirmer l'objectif de distribution.
   - Validation d'archive locale
   - App distribuable signée
   - Dépannage de notarization

2. Inspecter l'artefact.
   - Valider la structure de l'app bundle.
   - Vérifier les frameworks imbriqués, les helper tools et les entitlements.

3. Inspecter les prérequis de signature et de runtime.
   - Hardened runtime
   - Identité de signature
   - Signatures de code imbriquées
   - Entitlements requis

4. Expliquer la préparation ou l'échec de notarization.
   - Séparer les problèmes de packaging des symptômes de trust policy.
   - Pointer vers les commandes de validation de suivi minimales.

## Garde-fous

- Ne pas présenter la notarization comme requise pour un run de debug local ordinaire.
- Signaler quand l'artefact exporté réel manque et que l'analyse se base sur les réglages du projet.
- Garder des conseils concrets et vérifiables.

## Sortie attendue

Fournir :
- l'artefact ou les réglages inspectés
- si l'app semble prête pour la distribution
- le principal prérequis manquant ou mode d'échec
- la prochaine étape de validation ou de correction
