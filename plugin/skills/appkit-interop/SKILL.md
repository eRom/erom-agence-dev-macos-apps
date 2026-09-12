---
name: appkit-interop
description: Fait le pont entre SwiftUI et AppKit sur macOS, de façon volontairement étroite. À utiliser quand il faut implémenter des representables, accéder à NSWindow ou aux panels, gérer les menus, ou passer par la responder chain.
---

# AppKit Interop

## Démarrage rapide

Utiliser cette skill quand SwiftUI est presque suffisant, mais pas tout à fait, pour un comportement macOS natif. Garder le pont aussi petit et explicite que possible. SwiftUI doit rester la source de vérité dans la plupart des cas, pendant qu'AppKit prend en charge la partie impérative.

## Choisir le pont le plus petit possible

- Utiliser du SwiftUI pur quand le comportement recherché existe déjà dans les scenes, toolbars, commands, inspectors ou les contrôles standards.
- Utiliser `NSViewRepresentable` quand il faut une vue AppKit précise avec un cycle de vie léger.
- Utiliser `NSViewControllerRepresentable` quand il faut le cycle de vie d'un controller, de la délégation, ou de la coordination de présentation.
- Utiliser directement les hooks de fenêtre ou d'app AppKit quand il faut `NSWindow`, la responder chain, la validation de menu, des panels, ou un comportement au niveau app.

## Procédure

1. Nomme précisément l'écart de capacité.
   - Comportement de fenêtre
   - Comportement du système de texte
   - Validation de menu
   - Drag and drop
   - Panels d'ouverture/enregistrement de fichier
   - Contrôle du first responder

2. Choisir la frontière la plus petite qui règle le problème.
   - Éviter de porter tout un écran vers AppKit quand un seul contrôle ou coordinator suffirait.

3. Garder l'ownership explicite.
   - SwiftUI possède l'état de valeur, la sélection, et les modèles observables.
   - Les objets AppKit restent à l'intérieur du representable, du coordinator, ou de l'objet pont.

4. Exposer une interface étroite vers SwiftUI.
   - Des bindings pour l'état éditable
   - De petits callbacks pour les événements
   - Des services de pont ciblés, seulement quand c'est nécessaire

5. Valider les hypothèses de cycle de vie.
   - SwiftUI peut recréer les representables.
   - Les coordinators existent pour porter la glue de délégation et de target-action, pas comme une seconde architecture d'app.

## Références

- `references/representables.md` : choisir entre wrapper de vue et de view controller, plus les patterns de coordinator.
- `references/window-panels.md` : accès aux fenêtres, utility windows, et panels d'ouverture/enregistrement.
- `references/responder-menus.md` : first responder, routage de commandes, et validation de menu.
- `references/drag-drop-pasteboard.md` : pasteboard, URLs de fichiers, et cas limites du drag/drop desktop.

## Garde-fous

- Ne pas dupliquer la source de vérité entre SwiftUI et AppKit.
- Ne pas laisser `Coordinator` devenir un fourre-tout sans structure.
- Ne pas stocker d'instances `NSView` ou `NSWindow` à durée de vie longue de façon globale, sans raison d'ownership solide.
- Préférer un petit pont testé à une réécriture de la fonctionnalité en AppKit brut.
- Si un pattern peut rester entièrement dans `erom-dev-macos-apps:swiftui-patterns`, le laisser là.

## Sortie attendue

Fournir :
- la limitation SwiftUI précise franchie
- le type de pont minimal recommandé
- la frontière de flux de données entre SwiftUI et AppKit
- les risques de cycle de vie ou de validation à surveiller
