# Commands and Menus

## Intention

Utiliser ceci pour transformer des actions desktop en menu items, raccourcis clavier et comportement de scene focused.

## Patterns principaux

- Ajouter `commands` au niveau de la scene.
- Utiliser `CommandMenu` pour les actions spécifiques à l'app.
- Utiliser `CommandGroup` pour insérer, remplacer ou retirer des sections de menu.
- Utiliser `FocusedValue` ou le state de la scene pour rendre les commands sensibles au contexte.
- Associer les commands importantes à des raccourcis clavier et des affordances toolbar ou content visibles quand c'est pertinent.

## Exemple

```swift
@main
struct SampleApp: App {
  var body: some Scene {
    WindowGroup {
      EditorRootView()
    }
    .commands {
      CommandMenu("Document") {
        Button("New Note") { /* créer */ }
          .keyboardShortcut("n")

        Button("Toggle Inspector") { /* basculer */ }
          .keyboardShortcut("i", modifiers: [.command, .option])
      }
    }
  }
}
```

## Pièges

- Ne pas enregistrer le même raccourci à plusieurs endroits.
- Ne pas faire des commands le seul chemin découvrable pour une action critique.
- En cas de besoin de validation via la responder chain, d'un state de menu item custom ou d'un comportement de command spécifique à AppKit, utiliser la skill `erom-dev-macos-apps:appkit-interop`.
