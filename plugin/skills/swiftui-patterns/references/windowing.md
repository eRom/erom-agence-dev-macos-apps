# Windowing

## Intention

Utiliser ceci pour choisir le modèle de scene top-level d'une app macOS native.

## Choisir le type de scene délibérément

- Utiliser `WindowGroup(..., id:)` pour la fenêtre principale de l'app quand elle doit apparaître au lancement, surtout dans les apps qui ont aussi un `MenuBarExtra`.
- Utiliser `WindowGroup` pour toute scene pouvant avoir plusieurs instances indépendantes.
- Utiliser `Window` pour les utility windows singleton ou les surfaces secondaires focused. Dans les apps à dominante menu bar, `Window(...)` convient mieux aux fenêtres auxiliaires/à la demande et peut ne pas présenter la fenêtre principale automatiquement au lancement.
- Utiliser `Settings` pour les préférences. Ne pas enterrer les settings dans le flow de contenu principal.
- Utiliser `DocumentGroup` quand l'app est fondamentalement pilotée par des documents.

## Exemple : app principale plus utility window

Ce snippet montre seulement le câblage de la scene. Dans une vraie app non triviale,
garder l'app `@main` dans `App/<AppName>App.swift` et mettre `LibraryRootView`,
`InspectorRootView` et `SettingsView` dans des fichiers `Views/` dédiés.

```swift
@main
struct SampleApp: App {
  var body: some Scene {
    WindowGroup("Library", id: "library") {
      LibraryRootView()
    }

    Window("Inspector", id: "inspector") {
      InspectorRootView()
    }

    Settings {
      SettingsView()
    }
  }
}
```

## Ouvrir des fenêtres

- Utiliser `openWindow(id:)` quand une command, un toolbar item ou un bouton doit ouvrir une autre scene.
- Garder le state par fenêtre dans la scene ou `@SceneStorage`, pas dans un unique tas global.

## Pièges

- Éviter de modéliser chaque feature comme une destination pushée dans une seule fenêtre.
- Ne pas utiliser uniquement `Window(...)` pour la fenêtre de lancement principale d'une app menu-bar-plus-window, sauf si le comportement au lancement a été vérifié et qu'une fenêtre auxiliaire à la demande est voulue intentionnellement.
- Éviter le state singleton pour des sélections ou des drafts spécifiques à une fenêtre.
- En cas de besoin d'un contrôle bas niveau de titlebar, tabbing ou lifecycle de fenêtre, passer à la skill `erom-dev-macos-apps:appkit-interop`.
