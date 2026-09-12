# Windows et Panels

## Objectif

Utiliser cette référence quand les scenes SwiftUI ne suffisent pas pour le comportement de fenêtre ou de panel requis.

## Cas courants

- Accéder au `NSWindow` sous-jacent
- Configurer le comportement de la titlebar ou de la toolbar
- Présenter `NSOpenPanel` ou `NSSavePanel`
- Gérer des utility panels ou des floating windows

## Patterns de base

- Préférer d'abord `Window`, `WindowGroup`, et `openWindow` en SwiftUI.
- Utiliser AppKit seulement pour les fonctionnalités de fenêtre que SwiftUI n'expose pas proprement.
- Garder les panels d'ouverture/enregistrement de fichier derrière un petit service ou helper, plutôt que d'éparpiller leur configuration dans l'arbre de vues.

## Exemple : open panel

```swift
@MainActor
func chooseFile() -> URL? {
  let panel = NSOpenPanel()
  panel.canChooseFiles = true
  panel.canChooseDirectories = false
  panel.allowsMultipleSelection = false
  return panel.runModal() == .OK ? panel.url : nil
}
```

## Pièges

- Ne pas laisser des vues quelconques posséder des références `NSWindow` à durée de vie longue.
- Garder les floating panels et utility windows cohérents avec le modèle de scene.
- Si le comportement relève en fait juste des settings ou d'une scene secondaire, retourner vers `erom-dev-macos-apps:swiftui-patterns`.
