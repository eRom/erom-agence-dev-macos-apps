---
name: swiftpm-macos
description: Build, run et test de packages et executables macOS avec SwiftPM. À utiliser quand le dépôt est package-first ou n'a pas de projet Xcode.
---

# SwiftPM pour macOS

## Démarrage rapide

Utiliser cette skill quand `Package.swift` est le point d'entrée principal, ou
quand SwiftPM est le chemin le plus rapide vers un résultat reproductible.

## Procédure

1. Inspecter le package.
   - Lire `Package.swift`.
   - Identifier les produits executable, library et test.

2. Build avec SwiftPM.
   - Utiliser `swift build` par défaut.
   - Utiliser le mode release seulement quand l'utilisateur en a explicitement besoin.

3. Run le bon produit.
   - Utiliser `swift run <product>` quand un executable existe.
   - Si plusieurs executables existent, expliquer le choix par défaut.

4. Test ciblé.
   - Utiliser `swift test`.
   - Appliquer des filtres quand un target ou un cas de test précis est connu.

5. Résumer les échecs.
   - Résolution de module/import
   - Problème de graphe de packages ou de dépendance
   - Échec du linker
   - Échec au runtime
   - Régression de test

## Garde-fous

- Préférer SwiftPM à Xcode quand les deux existent et que le chemin package est clairement plus simple.
- Ne pas supposer qu'un app bundle existe dans un workflow package pur.
- Expliquer quand le package est library-only, donc non exécutable directement.

## Sortie attendue

Fournir :
- les produits du package trouvés
- la commande exécutée
- si le build, le run ou le test a réussi
- le principal blocage sinon
