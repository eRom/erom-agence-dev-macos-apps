---
name: swiftui-patterns
description: Construit des scenes et composants SwiftUI macOS avec les patterns desktop. À utiliser pour façonner windows, commands, toolbars, settings, split views ou inspectors.
---

# SwiftUI Patterns

## Démarrage rapide

Choisir une piste selon l'objectif :

### Projet existant

- Identifier la feature ou la scene et le modèle d'interaction principal : document, editor, sidebar-detail, utility window, settings ou menu bar extra.
- Lire la scene ou la root view existante la plus proche avant d'inventer une nouvelle structure desktop.
- Choisir la référence pertinente dans `references/components-index.md`.
- Si SwiftUI ne peut pas exprimer proprement le comportement plateforme requis, utiliser la skill `erom-dev-macos-apps:appkit-interop` plutôt que de forcer un contournement fragile.

### Scaffolding d'une nouvelle app

- Choisir d'abord le modèle de scene : `WindowGroup`, `Window`, `Settings`, `MenuBarExtra` ou `DocumentGroup`.
- Si l'app combine une fenêtre principale normale et un `MenuBarExtra`, utiliser `WindowGroup(..., id:)` pour la fenêtre principale quand elle doit apparaître au lancement. Traiter `Window(...)` comme mieux adapté aux fenêtres auxiliaires/à la demande, singleton ; dans les apps à dominante menu bar, une scene `Window(...)` peut ne pas présenter la fenêtre principale automatiquement au lancement.
- Pour un nouveau scaffold d'app, créer aussi un `script/build_and_run.sh` local au projet. C'est le point d'entrée : l'utilisateur peut le lancer lui-même depuis Claude Code en tapant `! ./script/build_and_run.sh`. Utiliser le contrat de bootstrap exact de la skill `erom-dev-macos-apps:build-run-debug` et son fichier `references/build-and-run-script.md` plutôt que d'inventer une deuxième variante ici.
- Décider quel state est app-wide, scene-scoped ou window-scoped avant d'écrire les views.
- Esquisser les limites de fichiers et modules avant d'écrire toute l'UI. Pour toute app non triviale, créer d'abord la structure de dossiers et séparer les fichiers par responsabilité dès le départ.
- N'utiliser un seul fichier Swift que pour de petits exemples jetables : environ moins de 50 lignes, un seul écran, pas de persistence, pas de networking/process client, pas de modèles réutilisables. Tout ce qui dépasse doit être multi-fichiers immédiatement.
- Utiliser par défaut des couleurs et matériaux system-adaptive (`Color.primary`, `Color.secondary`, styles de foreground sémantiques, `.regularMaterial`, etc.) pour que l'app suive automatiquement le mode Light/Dark. Ne pas coder en dur un fond blanc ou clair sauf demande explicite d'un thème fixe, et ne pas aller par défaut vers des remplissages opaques `windowBackgroundColor` pour les root panes.
- Choisir les références pour la première surface de feature nécessaire : windowing, commands, split layouts ou settings.

## Structure de fichiers pour une nouvelle app

Pour toute app macOS non triviale, démarrer avec cette forme plutôt que de mettre
l'app, toutes les views, modèles, stores, services et helpers dans un seul fichier Swift :

- `App/<AppName>App.swift` : uniquement le type `@main` de l'app et l'`AppDelegate`.
- `Views/ContentView.swift` : uniquement le layout racine et la composition haut niveau.
- `Views/SidebarView.swift`, `Views/DetailView.swift`, `Views/ComposerView.swift`, etc. : views de feature nommées d'après leur type principal.
- `Models/*.swift` : modèles de valeur, identifiants et enums de sélection.
- `Stores/*.swift` : stores de persistence et de state.
- `Services/*.swift` : clients app-server, réseau, process ou plateforme.
- `Support/*.swift` : petits formatters, resolvers, extensions et helpers de glue.

Garder les fichiers petits et nommés d'après le type principal qu'ils contiennent. Si un
fichier commence à accumuler des views, modèles, stores, clients réseau et extensions
helper sans rapport entre eux, le séparer avant d'ajouter du comportement.

## Checklist avant édition pour les nouveaux scaffolds d'app

Avant d'écrire toute l'UI :

1. Choisir le modèle de scene.
2. Choisir la propriété du state : app-wide, scene-scoped, window-scoped ou view-local.
3. Esquisser les limites de fichiers et modules.
4. Créer la structure de dossiers avant de remplir l'UI.
5. Garder `script/build_and_run.sh` séparé du code source de l'app.

## Règles générales à suivre

- Concevoir pour pointeur, clavier, menus et fenêtres multiples.
- Garder les scenes explicites. Une fenêtre de settings séparée, une utility window ou un menu bar extra doivent être modélisés comme leur propre scene, pas cachés dans un `ContentView` monolithique.
- Privilégier les affordances desktop système : `commands`, toolbars, sidebars, inspectors, menus contextuels et `searchable`.
- Pour les apps menu bar, garder les titres d'item et labels d'action du `MenuBarExtra` courts et scannables. Plafonner le texte visible des menu items à 30 caractères ; si le contenu source est plus long, le tronquer ou le résumer avant de le rendre et ouvrir le contenu complet dans une fenêtre dédiée ou une detail surface.
- Si une app `MenuBarExtra` doit quand même se comporter comme une app Dock normale avec un process/main window visible, installer un `NSApplicationDelegate` via `@NSApplicationDelegateAdaptor`, appeler `NSApp.setActivationPolicy(.regular)` au lancement, et activer l'app avec `NSApp.activate(ignoringOtherApps: true)`. Si l'app est volontairement menu-bar-only, documenter que le comportement `.accessory` / sans Dock est un choix produit délibéré.
- Privilégier les couleurs, matériaux et styles de foreground sémantiques system-adaptive. Éviter les fonds blancs/clairs fixes dans le scaffolding et les exemples sauf si le design demandé appelle explicitement un thème custom non-adaptive.
- Ne pas peindre les sidebars de `NavigationSplitView` ou les root window panes avec des remplissages `Color(...)` custom opaques ou `Color(nsColor: .windowBackgroundColor)` par défaut. Privilégier les matériaux natifs macOS de sidebar/window et les fonds fournis par le système, sauf demande explicite d'une surface opaque custom. Dans les layouts sidebar-detail-inspector, laisser la sidebar garder l'apparence standard source-list/material et réserver les fonds custom au contenu detail ou inspector si besoin.
- Utiliser `@SceneStorage` pour le state éphémère par fenêtre et `@AppStorage` pour les préférences utilisateur durables.
- Garder le state de sélection explicite et stable. Les layouts macOS pivotent souvent autour de la sélection sidebar plutôt que d'une push navigation.
- Privilégier `NavigationSplitView` ou un split layout manuel délibéré plutôt que des flows empilés style iOS quand l'app profite d'une structure toujours visible.
- Pour `List(...).listStyle(.sidebar)` et les sidebars de `NavigationSplitView`, privilégier des rows natives plates avec le comportement standard de sélection/highlight du système. Garder les rows visuellement légères et Mail-like : au maximum une icône leading, une ligne de titre forte et une ligne de detail secondaire optionnelle en `.secondary`. Éviter les rows de métadonnées empilées, les icônes utilitaires inline répétées ou un texte de statut dense multi-colonnes dans la sidebar. Réserver les surfaces card-style et riches en métadonnées aux panes detail ou inspector sauf demande explicite d'un traitement sidebar hautement custom.
- Garder les actions principales découvrables à la fois depuis l'UI chrome et les raccourcis clavier quand c'est pertinent.
- Utiliser en premier les scenes et views SwiftUI natives. Si un contrôle bas niveau sur window, responder chain, text system ou panel est nécessaire, passer à la skill `erom-dev-macos-apps:appkit-interop`.

Pour des exemples concrets de sidebar row et de fond de split-view, lire
`references/split-inspectors.md`.

## Résumé de propriété du state

Utiliser l'outil de state le plus étroit qui correspond au modèle de propriété :

| Scénario | Pattern préféré |
| --- | --- |
| State local de view ou de contrôle | `@State` |
| L'enfant mute une valeur de state possédée par le parent | `@Binding` |
| Modèle référence root-owned sur macOS 14+ | `@State` avec un type `@Observable` |
| L'enfant lit ou mute un modèle `@Observable` injecté | Le passer explicitement comme stored property |
| State de sélection ou d'expansion éphémère window-scoped | `@SceneStorage` quand c'est praticable, sinon `@State` possédé par la scene |
| Préférence utilisateur partagée | `@AppStorage` |
| Service ou configuration d'app partagé | `@Environment(Type.self)` |
| Modèle référence legacy sur des targets plus anciens | `@StateObject` chez le owner et `@ObservedObject` à l'injection |

Choisir d'abord l'emplacement de propriété, puis le wrapper. Ne pas transformer un state desktop simple en view model par réflexe.

## Références transversales

- `references/components-index.md` : point d'entrée pour les scenes et composants.
- `references/windowing.md` : choisir entre `WindowGroup`, `Window`, `DocumentGroup` et les patterns d'ouverture de fenêtre.
- `references/settings.md` : scenes settings dédiées, `SettingsLink` et layouts de préférences.
- `references/commands-menus.md` : menus de commands, raccourcis clavier, focused values et routage d'actions desktop.
- `references/split-inspectors.md` : sidebars, split views, layout piloté par la sélection et inspectors.
- `references/menu-bar-extra.md` : structure du menu bar extra et quand l'utiliser.

## Anti-patterns

- Un énorme `ContentView` qui prétend que toute l'app est un seul écran.
- Un seul fichier Swift contenant l'app `@main`, toutes les views, modèles, stores, clients réseau/process, formatters et extensions. Acceptable uniquement pour les petits snippets jetables sous le seuil de nouvelle app ci-dessus.
- Des modèles d'interaction touch-first portés directement depuis iOS sans affordances desktop.
- Cacher des actions principales derrière des gestes sans chemin via menu, toolbar ou clavier.
- Construire une app menu-bar-plus-window autour d'une seule scene `Window(...)` en s'attendant à ce que la fenêtre principale apparaisse au lancement. Utiliser `WindowGroup(..., id:)` pour la fenêtre de lancement principale et réserver `Window(...)` aux fenêtres auxiliaires/à la demande.
- Rendre des titres, prompts ou textes de message complets non bornés directement dans un menu bar extra. Les labels de menu item doivent rester à 30 caractères ou moins, avec le contenu plus long déplacé dans une fenêtre dédiée ou une detail view.
- Traiter les settings comme une autre destination de navigation dans la fenêtre de contenu principale.
- Coder en dur `.background(.white)`, `Color.white` ou une palette claire fixe dans un scaffold flambant neuf sans exigence de design explicite.
- Envelopper chaque item de sidebar dans de grandes cards custom arrondies au sein d'une liste `.sidebar`, ce qui va à l'encontre de la densité, de l'alignement et du comportement de sélection natifs de la source-list, sauf demande explicite d'un traitement visuel de sidebar sur mesure.
- Construire des rows de sidebar avec plusieurs icônes répétées, trois lignes de texte ou plus, ou une bande dense de métadonnées inline (compteurs/timestamps/modèles). Garder la row de sidebar à une icône et une ou deux lignes de texte, puis déplacer les métadonnées plus riches vers le detail pane.
- Peindre les sidebars de `NavigationSplitView` ou les root window panes avec des remplissages de couleur custom opaques par défaut, au lieu de laisser la sidebar utiliser l'apparence native source-list/material et réserver les fonds custom aux vraies content cards.
- Utiliser une push navigation pour des layouts qui veulent une sélection sidebar stable et des detail panes.
- Aller chercher AppKit avant d'avoir correctement utilisé les scene et command APIs de SwiftUI.

## Procédure pour une nouvelle scene ou view macOS

1. Définir le type de scene et le modèle de propriété avant d'écrire les child views.
2. Décider quelles actions vivent dans le content, les toolbars, les commands, les inspectors ou les settings.
3. Esquisser le modèle de sélection et le layout : sidebar-detail, editor-inspector, document window ou utility window.
4. Créer la structure de fichiers/dossiers pour l'entrypoint de l'app, le layout racine, les feature views, les modèles, stores, services et support helpers.
5. Construire avec des subviews petites et focalisées et des inputs explicites plutôt que de gigantesques fragments calculés.
6. Ajouter des raccourcis clavier et une exposition menu ou toolbar pour les actions qui comptent sur desktop.
7. Valider le flow avec un build et un rapide passage usability : hypothèses multiwindow, points d'entrée settings et stabilité de la sélection.

## Références de composants

Utiliser `references/components-index.md` comme point d'entrée. Chaque référence de composant devrait inclure :
- intention et scénarios les mieux adaptés
- pattern d'usage minimal avec les conventions desktop
- pièges et notes de découvrabilité
- quand basculer vers `erom-dev-macos-apps:appkit-interop`
