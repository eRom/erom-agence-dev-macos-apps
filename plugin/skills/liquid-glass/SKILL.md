---
name: liquid-glass
description: Implémente et passe en revue l'UI Liquid Glass de SwiftUI sur macOS. À utiliser lors de l'adoption du glass système, pour retirer du chrome personnalisé qui entre en conflit, ou pour construire des surfaces glass.
---

# Liquid Glass

## Vue d'ensemble

Utiliser cette skill pour amener une app SwiftUI macOS vers le design system macOS moderne avec le minimum de chrome personnalisé. Commencer par la structure d'app standard, les toolbars, le placement de la recherche, les sheets et les contrôles, puis ajouter du Liquid Glass personnalisé seulement là où l'app a besoin d'une surface distinctive.

Préférer le glass fourni par le système et les matériaux adaptatifs plutôt qu'un flou fait maison, des fonds opaques, ou des habillages de toolbar/sidebar personnalisés. Auditer l'UI existante pour repérer les fills, scrims et clippings superflus avant d'ajouter d'autres effets.

## Procédure

1. Lire la scene ou la root view concernée et identifier le pattern structurel : `NavigationSplitView`, `TabView`, présentation de sheet, layout detail/inspector, toolbar, ou contrôles flottants personnalisés.
2. Retirer les fonds personnalisés ou les couches d'assombrissement derrière les sheets système, les sidebars et les toolbars, sauf si le produit en a explicitement besoin. Ils peuvent masquer le Liquid Glass et interférer avec l'effet automatique de scroll-edge.
3. Mettre d'abord à jour la structure et les contrôles SwiftUI standards.
4. Ajouter des surfaces `glassEffect` personnalisées seulement pour l'UI spécifique à l'app que les contrôles standards ne couvrent pas.
5. Valider que le groupement du glass, les transitions, le traitement des icônes, et l'activation du foreground sont visuellement cohérents et restent utilisables au pointeur et au clavier.
6. Si le changement d'UI affecte aussi le comportement de lancement d'une app GUI SwiftPM, utiliser `erom-dev-macos-apps:build-run-debug` pour que l'app tourne comme un bundle `.app` au foreground plutôt que comme un exécutable brut.

## Structure de l'app

- Préférer `NavigationSplitView` pour les layouts macOS pilotés par une hiérarchie. Laisser la sidebar utiliser le matériau Liquid Glass système au lieu de peindre par-dessus.
- Pour une illustration hero ou un média large à côté d'une sidebar flottante, utiliser `backgroundExtensionEffect` pour que le visuel puisse s'étendre au-delà de la safe area sans se faire clipper.
- Garder les inspectors visuellement associés à la sélection courante, et éviter de leur donner un fond personnalisé plus lourd que le contenu qu'ils inspectent.
- Si l'app utilise des tabs, garder `TabView` pour les sections principales persistantes et préserver l'état de navigation local de chaque tab.
- Ne pas forcer le comportement de minimisation de tab bar/accessory propre à l'iPhone sur une app Mac. Sur macOS, préférer une toolbar top classique et un placement natif des tabs/de la recherche.
- Si une sheet utilise déjà `presentationBackground` juste pour imiter un matériau dépoli, envisager de le retirer et de laisser le nouveau matériau système s'afficher.
- Pour des transitions de sheet qui doivent visuellement partir d'un bouton de toolbar, faire de l'item présentant la source d'une navigation zoom transition, et marquer le contenu de la sheet comme la destination.

## Toolbars

- Considérer que les items de toolbar sont rendus sur une surface Liquid Glass flottante et sont groupés automatiquement.
- Utiliser `ToolbarSpacer` pour communiquer le groupement :
  - un espacement fixe pour séparer des actions liées en un groupe distinct,
  - un espacement flexible pour éloigner une action leading d'un groupe trailing.
- Utiliser `sharedBackgroundVisibility` quand un item doit se détacher du fond glass partagé, par exemple un item profil/avatar.
- Ajouter `badge` au contenu d'un item de toolbar pour des indicateurs de notification ou de statut.
- S'attendre à un rendu d'icônes monochrome dans davantage de contextes de toolbar. N'utiliser `tint` que pour porter un sens sémantique, comme une action primaire ou un état d'alerte, jamais pour de la pure décoration.
- Si le contenu sous une toolbar a des couches supplémentaires d'assombrissement, de flou, ou de fond personnalisé, les retirer avant de juger le nouvel effet automatique de scroll-edge.
- Pour des fenêtres denses avec beaucoup d'éléments flottants, ajuster le traitement de scroll-edge du contenu avec `scrollEdgeEffectStyle` plutôt que de construire un fond de barre personnalisé.

## Recherche

- Pour un champ de recherche qui s'applique à toute une hiérarchie split-view, attacher `searchable` au `NavigationSplitView`, pas juste à une colonne.
- Quand la recherche est secondaire et qu'une affordance compacte est préférable, utiliser `searchToolbarBehavior` plutôt que de bricoler un bouton de toolbar et un champ séparé.
- Pour une page de recherche dédiée dans une app multi-tabs, attribuer le rôle de recherche à un tab et placer `searchable` sur le `TabView`.
- Rendre la majorité du contenu de l'app découvrable depuis la recherche quand le champ vit dans l'emplacement top-trailing de la toolbar.
- Sur iPad et Mac, s'attendre à ce que le tab de recherche dédié affiche un champ centré au-dessus des suggestions de navigation, plutôt qu'une barre de recherche en bas.

## Contrôles

- Préférer les contrôles SwiftUI standards avant de créer des composants glass personnalisés.
- S'attendre à ce que les boutons bordered adoptent par défaut une forme capsule aux tailles plus grandes. Sur macOS, les tailles de contrôle mini/small/medium préservent une forme rectangle arrondi pour des layouts plus denses.
- Utiliser `buttonBorderShape` quand la forme d'un bouton doit être explicite.
- Utiliser `controlSize` pour préserver la densité dans les inspectors et les popovers, et réserver les tailles extra-large aux actions vraiment prominentes.
- Utiliser les styles de bouton glass et glass-prominent du système pour les actions primaires plutôt que de recréer à la main un fond de bouton translucide.
- Pour des sliders à valeurs discrètes, passer `step` pour obtenir des graduations automatiques, ou fournir des graduations spécifiques dans une closure `ticks`.
- Pour des sliders qui doivent s'étendre à gauche et à droite autour d'une valeur de référence, fixer `neutralValue`.
- Utiliser `Label` ou les initialiseurs de contrôle standards pour les items de menu, afin que les icônes soient placées de façon cohérente sur le bord leading, quelle que soit la plateforme.
- Pour des formes personnalisées qui doivent s'aligner de façon concentrique avec une sheet, une carte, ou un coin de fenêtre, utiliser une forme de rectangle concentrique avec la configuration de coin `containerConcentric` plutôt que de deviner un radius.

## Liquid Glass personnalisé

- Utiliser `glassEffect` pour les surfaces glass personnalisées. La forme par défaut est de type capsule, et les foregrounds de texte sont rendus automatiquement vibrants et lisibles face à un contenu changeant en dessous.
- Passer une forme explicite à `glassEffect` quand une capsule n'est pas adaptée.
- N'ajouter `tint` que quand la couleur porte un sens, comme un statut ou un appel à l'action.
- Utiliser `glassEffect(... .interactive())` pour les contrôles ou containers personnalisés avec des éléments interactifs, afin qu'ils scalent, rebondissent et scintillent comme le glass système.
- Regrouper les éléments glass personnalisés proches dans un seul `GlassEffectContainer`. C'est une règle de correction visuelle, pas seulement d'organisation : des containers séparés ne peuvent pas échantillonner le glass l'un de l'autre, ce qui peut produire une réfraction incohérente.
- Utiliser `glassEffectID` avec un `@Namespace` local quand des éléments glass correspondants doivent morpher entre états réduit et étendu.

## Checklist de revue

- Les structures et contrôles standards ont été mis à jour en premier, avant l'ajout de glass personnalisé.
- Les fonds opaques, scrims sombres, et fills personnalisés de toolbar/sheet qui entrent en conflit avec le matériau système ont été retirés, sauf besoin intentionnel.
- `searchable` est attaché au bon niveau de container pour la portée de recherche voulue.
- Le groupement de toolbar utilise `ToolbarSpacer`, `sharedBackgroundVisibility`, et `badge` plutôt qu'un chrome fait main au cas par cas.
- La teinte des icônes est sémantique, pas décorative.
- Les éléments glass personnalisés proches les uns des autres partagent un `GlassEffectContainer`.
- Les transitions de glass qui morphent utilisent `glassEffectID` avec un namespace et une identité stable.
- Toute app GUI SwiftPM utilisée pour tester le résultat est lancée comme un bundle `.app`, pas comme un exécutable brut.

## Garde-fous

- Ne pas reconstruire les sidebars, toolbars, sheets ou contrôles système depuis zéro si les API SwiftUI standards fournissent déjà le comportement macOS moderne.
- Ne pas appliquer de fond opaque personnalisé derrière une sidebar de `NavigationSplitView`, une toolbar système, ou une sheet juste parce qu'une ancienne version en avait besoin.
- Ne pas éparpiller des éléments glass liés entre eux sur plusieurs `GlassEffectContainer`.
- Ne pas teinter chaque icône ou surface glass juste pour varier visuellement.
- Ne pas supposer qu'un comportement de tab/recherche façon iPhone est la bonne réponse sur macOS. Préférer un placement desktop natif de toolbar, split-view, et inspector.
- Ne pas laisser une app GUI SwiftPM se lancer comme un exécutable nu lors de la revue du comportement Liquid Glass ; une activation du foreground manquante peut faire ressembler un bug de design à un bug de rendu.

## Quand utiliser d'autres skills

- Utiliser `erom-dev-macos-apps:swiftui-patterns` quand la question principale porte sur l'architecture de scene, le layout sidebar/detail, les commands, ou les settings plutôt que sur le traitement spécifique à Liquid Glass.
- Utiliser `erom-dev-macos-apps:view-refactor` quand le problème principal est la structure de fichiers, l'ownership d'état, et l'extraction de grosses vues avant des changements de design.
- Utiliser `erom-dev-macos-apps:appkit-interop` quand le design exige un comportement de fenêtre, de panel, de responder chain, ou de contrôle AppKit uniquement.
- Utiliser `erom-dev-macos-apps:build-run-debug` pour lancer, vérifier, ou inspecter les logs de l'app après la mise à jour visuelle.
