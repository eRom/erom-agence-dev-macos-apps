---
name: signing-entitlements
description: Inspecte les problèmes de signature macOS, d'entitlements et de Gatekeeper. À utiliser pour diagnostiquer un échec de codesign, de sandbox, de hardened runtime ou de confiance système.
---

# Signature et entitlements

## Démarrage rapide

Utiliser cette skill quand l'échec sent le codesigning plutôt que la
compilation : refus de lancement, entitlement manquant, signature invalide,
incohérence de sandbox, confusion de hardened runtime, ou rejet par une
trust policy.

## Procédure

1. Inspecter le bundle ou le binaire.
   - Localiser le `.app` ou l'executable.
   - Identifier le binaire principal dans `Contents/MacOS/`.

2. Lire les détails de signature.
   - Utiliser `codesign -dvvv --entitlements :- <path>`.
   - Utiliser `spctl -a -vv <path>` quand le comportement Gatekeeper compte.
   - Utiliser `plutil -p` pour inspecter les entitlements ou l'Info.plist.

3. Classer l'échec.
   - Non signé ou signé ad hoc
   - Mauvaise identité
   - Incohérence d'entitlement
   - Problème de hardened runtime
   - Problème d'App Sandbox
   - Problème de signature de code imbriquée
   - Prérequis de distribution/notarization manquant

4. Expliquer le chemin de correction minimal.
   - Dire exactement ce qui ne va pas.
   - Montrer la plus courte séquence de commandes de validation ou de réparation.
   - Distinguer les problèmes de développement local des problèmes de distribution.

## Commandes de référence

- `codesign -dvvv --entitlements :- <app-or-binary>`
- `spctl -a -vv <app-or-binary>`
- `security find-identity -p codesigning -v`
- `plutil -p <path-to-entitlements-or-plist>`

## Garde-fous

- Ne jamais inventer un entitlement manquant.
- Ne pas confondre notarization et signature de debug locale.
- Si le vrai problème est un build setting ou un provisioning profile, le dire directement.

## Sortie attendue

Fournir :
- l'artefact inspecté
- l'état de signature dans lequel il se trouve
- la classe d'échec exacte
- la séquence minimale de correction ou de validation
