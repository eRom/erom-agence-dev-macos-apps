---
name: test-triage
description: Triage des tests macOS sous Xcode et SwiftPM. À utiliser pour restreindre des échecs, expliquer des assertions ou des crashs, ou séparer un problème de setup d'une vraie régression.
---

# Triage des tests

## Démarrage rapide

Utiliser cette skill pour lancer d'abord le plus petit scope de test pertinent,
classer précisément les échecs, et éviter de traiter chaque échec de test
comme un bug produit.

## Procédure

1. Détecter le harness de test.
   - Utiliser `xcodebuild test` pour les projets basés sur Xcode.
   - Utiliser `swift test` pour les packages SwiftPM.

2. Restreindre le scope.
   - Si l'utilisateur a donné un target, un produit ou un filtre de test, l'utiliser.
   - Sinon, préférer le plus petit target probablement en échec avant une suite complète.

3. Classer le résultat.
   - Échec de build
   - Échec d'assertion
   - Crash ou signal
   - Timing async ou flake
   - Problème d'environnement ou de fixture
   - Entitlement manquant ou problème de host app

4. Relancer intelligemment.
   - Utiliser des reruns ciblés quand un cas précis échoue.
   - Éviter de perdre du temps sur des reruns de suite complète sans information nouvelle.

5. Résumer clairement.
   - La commande exécutée
   - Les tests en échec
   - Le type d'échec
   - La meilleure prochaine étape de preuve ou de correction

## Garde-fous

- Distinguer les échecs de compilation des échecs d'exécution de test.
- Signaler quand un test semble supposer un comportement iOS-only ou simulator-only.
- Marquer les flakes probables comme tels plutôt que de surestimer la confiance.

## Sortie attendue

Fournir :
- la commande utilisée
- le plus petit scope en échec
- la principale catégorie d'échec
- une explication concise de la cause probable
- la prochaine étape de rerun ou de correction
