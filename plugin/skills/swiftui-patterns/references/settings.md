# Settings

## Intention

Utiliser ceci pour construire une fenêtre de settings macOS native avec SwiftUI.

## Patterns principaux

- Déclarer une scene `Settings` dédiée dans l'app.
- Garder le contenu des settings dans une root view séparée.
- Utiliser `@AppStorage` pour les préférences utilisateur qui doivent persister.
- Privilégier des tabs, sections ou un layout de settings en split plutôt qu'une push navigation profonde.
- Utiliser `SettingsLink` ou `OpenSettingsAction` pour les points d'entrée in-app.

## Exemple

Ce snippet montre seulement le câblage de la scene. Dans une vraie app non triviale,
garder l'app `@main` dans `App/<AppName>App.swift` et mettre le contenu des settings
dans un fichier de view dédié comme `Views/SettingsView.swift`.

```swift
@main
struct SampleApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }

    Settings {
      SettingsView()
    }
  }
}

struct SettingsView: View {
  @AppStorage("showSidebarIcons") private var showSidebarIcons = true

  var body: some View {
    TabView {
      Form {
        Toggle("Show Sidebar Icons", isOn: $showSidebarIcons)
      }
      .tabItem { Label("General", systemImage: "gearshape") }
    }
    .frame(width: 460, height: 260)
    .scenePadding()
  }
}
```

## Pièges

- Ne pas réutiliser un écran de settings iOS plein écran, sauf si l'app est vraiment un portage direct style Catalyst.
- Garder les rows de settings simples et accessibles.
- Si les settings nécessitent des panels custom, des responders ou une intégration first-responder, utiliser la skill `erom-dev-macos-apps:appkit-interop`.
