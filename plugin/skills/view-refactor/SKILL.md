---
name: view-refactor
description: Refactore les views et scenes SwiftUI macOS vers une structure stable. À utiliser pour découper de grosses views, resserrer le state de scene ou restreindre les échappées vers AppKit.
---

# View Refactor

## Vue d'ensemble

Cette skill refactore les views macOS vers des types de scene et de view petits, explicites et stables.
Elle privilégie par défaut SwiftUI natif pour le layout, la sélection, les commands et les settings,
et ne va chercher AppKit qu'aux marges étroites où le comportement desktop l'exige vraiment.

## Directives principales

### 1) Modéliser les scenes explicitement

- Découper l'app en racines de scene significatives : fenêtre principale, settings, utility windows, inspectors ou menu bar extras.
- Ne pas laisser une seule root view géante posséder silencieusement toute surface desktop.

### 2) Garder une forme de fichier prévisible

- Suivre cet ordre sauf si le fichier a déjà une convention locale plus forte :
- Environment
- `private`/`public` `let`
- `@State` / autres stored properties
- `var` calculée (non-view)
- `init`
- `body`
- view builders calculés / autres helpers de view
- fonctions helper / async

### 2b) Séparer les fichiers par responsabilité

- Pour les apps non triviales, ne pas garder toute l'app, les views, les modèles, les stores, les clients réseau, les clients process et les helpers dans un seul fichier Swift.
- Accepter un seul fichier Swift uniquement pour de petits exemples jetables : environ moins de 50 lignes, un seul écran, pas de persistence, pas de networking/process client, pas de modèles réutilisables.
- Utiliser `App/<AppName>App.swift` uniquement pour l'app `@main` et l'`AppDelegate`.
- Garder `Views/ContentView.swift` focalisé sur le layout racine et la composition ; déplacer l'UI de feature dans des fichiers comme `Views/SidebarView.swift`, `Views/DetailView.swift` et `Views/ComposerView.swift`.
- Déplacer les value types et les enums de sélection dans `Models/*.swift`, les stores dans `Stores/*.swift`, les clients app-server/réseau/process dans `Services/*.swift`, et les petits formatters/resolvers/extensions dans `Support/*.swift`.
- Garder les fichiers petits et nommés d'après le type principal qu'ils contiennent.

### 3) Privilégier des types de subview dédiés plutôt que de nombreux fragments `some View` calculés

- Extraire les sections desktop significatives comme les sidebar rows, detail panels, inspectors ou contenu de toolbar dans des subviews focalisées.
- Garder les helpers `some View` calculés petits et rares.
- Passer des données, bindings et actions explicites aux subviews plutôt que de leur transmettre tout le modèle de scene.

### 4) Garder la sélection et le layout stables

- Privilégier un unique layout de split ou de fenêtre stable avec des conditionnels locaux à l'intérieur.
- Éviter de basculer entre des racines radicalement différentes au niveau top-level quand la sélection change.
- Laisser le layout constant ; laisser le state piloter le contenu à l'intérieur.

### 5) Extraire commands, toolbars et actions hors de `body`

- Ne pas enterrer de la logique de bouton non triviale inline.
- Ne pas mélanger le routage de commands, le state de menu et le layout dans le même bloc quand on peut les nommer clairement.
- Garder `body` lisible comme de l'UI, pas comme un view controller desktop.

### 6) Utiliser scene et app storage intentionnellement

- Utiliser `@SceneStorage` pour le state éphémère par fenêtre quand ça aide vraiment à restaurer la scene.
- Utiliser `@AppStorage` pour les préférences durables, pas pour des toggles UI transitoires qui ne comptent que dans une fenêtre.
- Garder le state possédé par la scene proche de la racine de scene.

### 7) Garder les échappées AppKit étroites

- Si un representable ou un bridge `NSWindow` existe, l'isoler derrière un petit wrapper ou helper.
- Ne pas laisser des références AppKit se répandre dans des views SwiftUI sans rapport.
- Si le bridge commence à posséder la feature, réévaluer l'architecture.

### 8) Usage de l'observation

- Pour les types référence `@Observable` sur les targets macOS modernes, les stocker comme `@State` dans la view propriétaire.
- Passer les observables explicitement aux enfants.
- Sur des deployment targets plus anciens, retomber sur `@StateObject` et `@ObservedObject` si besoin.

## Procédure

1. Identifier la limite de scene actuelle et si le fichier essaie d'en faire trop.
2. Réordonner le fichier dans une structure top-to-bottom prévisible.
3. Extraire les sections spécifiques au desktop dans des types de subview dédiés.
4. Stabiliser le layout racine autour de la sélection, des scenes et des commands plutôt que d'un branchement top-level.
5. Déplacer la logique d'action, le routage de commands et le comportement de toolbar dans des helpers nommés ou des types séparés.
6. Resserrer tout bridge AppKit pour que la marge impérative reste petite et explicite.
7. Garder le comportement intact sauf si la demande appelle explicitement des changements structurels et comportementaux en même temps.

## Checklist de refactor

- Découper les fichiers de view surdimensionnés avant d'ajouter plus d'UI.
- Déplacer les modèles purs, identifiants et enums de sélection hors des fichiers de view.
- Déplacer le code `Process`, `URLSession`, app-server et client plateforme hors des views SwiftUI vers `Services/`.
- Garder `AppDelegate` et l'entrypoint d'app `@main` minimaux.
- Build après chaque découpage majeur pour que les erreurs de compilation restent locales.

## Odeurs courantes

- Une root view qui mélange scaffolding de fenêtre, settings, code de toolbar, gestion de commands et layout de detail.
- Un seul fichier d'app qui mélange entrypoint d'app, layout racine, feature views, modèles, stores, clients de service et extensions support.
- Une push navigation style iOS forcée dans un problème sidebar-detail Mac.
- Plusieurs booléens pour des inspectors, sheets ou utility windows mutuellement exclusifs.
- Des objets AppKit passés à travers de nombreuses couches SwiftUI sans raison de propriété claire.
- De gros fragments de view calculés tenant lieu de vraies subviews.

## Notes

- Un bon refactor macOS devrait rendre évidents la structure de scene, le flow de sélection et la propriété des commands.
- Quand le problème est fondamentalement un pattern desktop manquant, utiliser la skill `erom-dev-macos-apps:swiftui-patterns`.
- Quand le problème est fondamentalement une frontière avec AppKit, utiliser la skill `erom-dev-macos-apps:appkit-interop`.
