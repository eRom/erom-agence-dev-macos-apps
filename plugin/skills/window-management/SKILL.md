---
name: window-management
description: Personnalise les windows et le comportement de scene SwiftUI sur macOS. À utiliser pour ajuster le chrome de fenêtre, les zones de drag, le placement, la restoration, le comportement au lancement, ou les borderless windows.
---

# Window Management

## Vue d'ensemble

Utiliser cette skill pour adapter chaque fenêtre SwiftUI à son rôle. Commencer par identifier quelle scene possède la fenêtre (`Window`, `WindowGroup`, ou une scene utility dédiée), puis personnaliser la zone toolbar/titre, le matériau de fond, le comportement de resize et de restoration, et le placement initial ou zoomé.

Préférer les modificateurs de scene et de window aux ponts AppKit improvisés quand SwiftUI offre directement le comportement voulu. Garder chaque fenêtre construite pour son usage : une fenêtre principale de navigation, une fenêtre About, et une fenêtre de lecteur média veulent en général un chrome, une resizability, une restoration, et des règles de placement différents.

Ces API sont des personnalisations de window/scene SwiftUI de macOS 15+. Pour des cibles de déploiement plus anciennes, s'attendre à devoir passer par plus de bridging AppKit ou de garde-fous de disponibilité.

## Procédure

1. Inspecter la déclaration de scene concernée et classer le rôle de la fenêtre : navigation principale de l'app, utility inspector/detail, fenêtre About/support, fenêtre de lecture média, fenêtre welcome, ou surface borderless personnalisée.
2. Ajuster la présentation de la toolbar et du titre pour correspondre au contenu.
3. Si le fond de la toolbar ou la toolbar entière est masqué, s'assurer que la fenêtre garde une zone de drag utilisable.
4. Affiner le comportement de la fenêtre pour ce rôle : disponibilité de la minimisation, restoration, attentes de resize, et si la fenêtre doit apparaître au lancement.
5. Fixer le placement par défaut des fenêtres nouvellement ouvertes et le placement idéal pour le comportement de zoom, quand le contenu et la taille d'écran comptent.
6. Builder et lancer l'app avec `erom-dev-macos-apps:build-run-debug` pour vérifier le résultat dans un vrai bundle `.app` au foreground.
7. Si les modificateurs de scene/window SwiftUI ne suffisent pas, passer à `erom-dev-macos-apps:appkit-interop` pour un pont `NSWindow` étroit, plutôt que de disperser AppKit dans l'arbre de vues.

## Toolbar et titre

- Utiliser `.toolbar(removing: .title)` quand le titre de la fenêtre doit rester associé à la fenêtre pour l'accessibilité et les menus, mais sans être visiblement dessiné dans la title bar.
- Utiliser `.toolbarBackgroundVisibility(.hidden, for: .windowToolbar)` quand un média ou un contenu hero doit s'étendre visuellement jusqu'au bord supérieur de la fenêtre.
- Si la fenêtre a toujours besoin des contrôles close/minimize/full-screen, ne retirer que le titre et le fond de toolbar. Si la toolbar doit disparaître entièrement, utiliser plutôt `.toolbarVisibility(.hidden, for: .windowToolbar)`.
- Retirer les fonds de toolbar personnalisés et les remplissages de titlebar peints à la main avant de superposer les nouvelles API de toolbar SwiftUI.
- Garder le titre logique de la fenêtre porteur de sens même s'il est masqué ; le système peut toujours l'utiliser pour l'accessibilité et les items de menu. Ce sont des changements uniquement visuels.

## Zones de drag

- Si un fond de toolbar est masqué ou si la toolbar est entièrement retirée, utiliser `WindowDragGesture()` pour étendre la zone draggable dans le contenu.
- Attacher le geste à un overlay transparent ou à une zone d'en-tête non interactive qui ne vole pas les gestes aux vrais contrôles.
- Pour un lecteur média avec des contrôles de lecture personnalisés, insérer l'overlay de drag entre le contenu vidéo et les contrôles, pour qu'AVKit ou les contrôles de transport continuent de recevoir les événements.
- Associer le geste de drag à `.allowsWindowActivationEvents(true)` pour qu'un clic suivi immédiatement d'un drag sur une fenêtre en arrière-plan l'active et la déplace quand même.

## Fond et matériaux

- Utiliser `.containerBackground(.thickMaterial, for: .window)` quand une utility window ou une fenêtre About doit remplacer le fond de fenêtre par défaut par un matériau dépoli discret.
- Préférer les matériaux système aux couleurs translucides codées en dur pour les fenêtres stylisées.
- Utiliser cette approche surtout pour les utility windows à contenu fixe, où un fond plus doux fait partie du design.

## Comportement de fenêtre

- Utiliser `.windowMinimizeBehavior(.disabled)` pour les utility windows toujours accessibles, comme une fenêtre About personnalisée, où minimiser apporte peu de valeur.
- Désactiver le contrôle de zoom vert via un dimensionnement fixe ou des contraintes de fenêtre quand le contenu de la fenêtre n'a qu'une seule taille prévue.
- Utiliser `.restorationBehavior(.disabled)` pour les fenêtres qui ne doivent pas se rouvrir au lancement suivant, comme les panels About, les fenêtres de support/info transitoires, ou les surfaces welcome de premier lancement.
- Garder la restoration d'état activée pour les fenêtres principales de document ou de navigation, quand rouvrir la taille et la position précédentes est souhaitable.
- Par défaut, SwiftUI respecte le réglage système de restoration d'état de macOS. N'utiliser `restorationBehavior(...)` que quand une fenêtre spécifique doit intentionnellement s'écarter de ce comportement système.
- Utiliser `.defaultLaunchBehavior(.presented)` pour les fenêtres qui doivent apparaître en premier au lancement, comme une fenêtre welcome, et choisir ce comportement intentionnellement plutôt que de s'appuyer sur des effets de bord de l'ordre de création des scenes.

## Placement de fenêtre

- Utiliser `.defaultWindowPlacement { content, context in ... }` pour contrôler la taille initiale et la position optionnelle des fenêtres nouvellement ouvertes.
- Dans la closure de placement, appeler `content.sizeThatFits(.unspecified)` pour obtenir la taille idéale du contenu.
- Lire `context.defaultDisplay.visibleRect` pour obtenir la région utilisable de l'écran après prise en compte de la menu bar et du Dock.
- Retourner `WindowPlacement(size: size)` avec une taille clampée à la visible rect quand le média ou le document peut être plus grand que l'écran. Si aucune position n'est fournie, la fenêtre est centrée par défaut.
- Utiliser `.windowIdealPlacement { content, context in ... }` pour contrôler ce qui se passe quand l'utilisateur choisit Zoom dans le menu Window ou fait Option-clic sur le bouton vert de la toolbar. Pour les fenêtres média, préserver le ratio d'aspect et grandir jusqu'à la plus grande taille qui tient dans l'écran.
- Traiter le placement par défaut et le placement idéal comme deux politiques séparées :
  - le placement par défaut contrôle où une nouvelle fenêtre apparaît en premier,
  - le placement idéal contrôle jusqu'à quelle taille une fenêtre zoomée doit grandir.
- Toujours prendre en compte les écrans externes et les écrans tournés/étroits en dimensionnant des fenêtres de lecteur ou de document à partir des dimensions du contenu.

## Windows borderless et spécialisées

- Utiliser `.windowStyle(.plain)` pour des fenêtres borderless ou au chrome fortement personnalisé, mais s'assurer que le contenu fournit toujours une affordance de drag/déplacement claire et un contexte visible.
- Pour un lecteur borderless, un HUD, ou une fenêtre welcome, décider en amont si perdre les affordances standards de la titlebar en vaut la peine face à la présentation personnalisée.
- Garder un chemin de retour clair vers la gestion de fenêtre classique si le style plain rend la fenêtre invisible ou difficile à déplacer.

Pour des exemples concrets de modificateurs de fenêtre, lire `references/api-snippets.md`.

## Checklist de revue

- Le type de scene correspond au rôle et au cycle de vie de la fenêtre.
- Les titres masqués laissent quand même un titre logique porteur de sens pour l'accessibilité et les menus.
- Le retrait du fond de toolbar est intentionnel et ne nuit pas à la lisibilité de la titlebar ni au placement des contrôles de fenêtre.
- Les fenêtres avec toolbar masquée ou retirée gardent une zone de drag fiable et supportent l'activation par clic-puis-drag depuis l'arrière-plan.
- Les utility windows ont un comportement de restoration/minimisation qui correspond à leur usage.
- Les overrides de restoration ne sont utilisés que quand une scene doit intentionnellement s'écarter du réglage système de l'utilisateur.
- Le placement par défaut et idéal utilisent `content.sizeThatFits(.unspecified)` et `context.defaultDisplay.visibleRect` quand la taille du contenu/de l'écran compte.
- Les fenêtres média préservent le ratio d'aspect et tiennent sur des écrans petits ou tournés.
- Les fenêtres borderless gardent une affordance de drag/déplacement utilisable.

## Garde-fous

- Ne pas utiliser `.toolbar(removing: .title)` juste pour masquer un titre oublié. Garder le titre de fenêtre sous-jacent porteur de sens.
- Ne pas masquer le fond de toolbar ou toute la toolbar sans remplacer l'affordance de drag perdue.
- Ne pas désactiver la restoration sur la fenêtre principale de document/navigation, sauf si l'utilisateur veut explicitement une app qui redémarre à zéro à chaque lancement.
- Ne pas coder en dur la taille d'un seul écran et ne pas supposer une configuration mono-écran en dimensionnant des fenêtres de lecteur.
- Ne pas sauter directement à la mutation de `NSWindow` avant d'avoir vérifié si `.windowMinimizeBehavior`, `.restorationBehavior`, `.defaultWindowPlacement`, `.windowIdealPlacement`, `.windowStyle`, ou `.defaultLaunchBehavior` règlent déjà le problème.
- Ne pas laisser une fenêtre borderless plain sans aucun chemin de drag ou de fermeture évident.

## Quand utiliser d'autres skills

- Utiliser `erom-dev-macos-apps:swiftui-patterns` pour une architecture plus large de scene, commands, settings, sidebar, et inspector.
- Utiliser `erom-dev-macos-apps:liquid-glass` quand la question principale est le traitement visuel macOS moderne, Liquid Glass, ou l'adoption de matériau système.
- Utiliser `erom-dev-macos-apps:appkit-interop` si un comportement de fenêtre personnalisé exige vraiment `NSWindow`, `NSPanel`, ou un contrôle de responder chain.
- Utiliser `erom-dev-macos-apps:build-run-debug` pour lancer et vérifier les fenêtres résultantes.
