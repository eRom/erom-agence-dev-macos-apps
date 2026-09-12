# Menu Bar Extra

## Intention

Utiliser ceci quand l'app vit principalement dans la menu bar macOS plutôt que dans une fenêtre traditionnelle toujours ouverte.

## Patterns principaux

- Utiliser `MenuBarExtra` pour les utilitaires légers, les indicateurs de statut et les actions rapides.
- Si l'app a aussi une fenêtre principale qui doit apparaître au lancement, définir cette scene avec `WindowGroup(..., id:)` et utiliser `Window(...)` uniquement pour les fenêtres auxiliaires/à la demande.
- Si l'app menu bar doit quand même apparaître dans le Dock et s'activer comme une app normale, installer un app delegate avec `@NSApplicationDelegateAdaptor`, appeler `NSApp.setActivationPolicy(.regular)` au lancement, puis `NSApp.activate(ignoringOtherApps: true)`.
- Si l'app est volontairement menu-bar-only, documenter explicitement que le comportement `.accessory` / sans Dock est un comportement produit attendu, pas un bug de lancement.
- Garder le contenu du menu concis et orienté action.
- Garder chaque label de menu item visible à 30 caractères ou moins. Si le contenu source peut être plus long, dériver un titre d'affichage court et ouvrir le texte complet dans une fenêtre séparée ou un detail pane.
- Si l'app a des workflows plus profonds, ouvrir une fenêtre dédiée depuis le menu bar extra plutôt que de tout entasser dans le menu.

## Exemple

Ce snippet montre seulement le câblage de la scene. Dans une vraie app non triviale,
garder l'app `@main` et l'`AppDelegate` dans `App/<AppName>App.swift`, et mettre la
menu bar, le contenu racine et les modèles/services support dans des fichiers séparés
nommés d'après leur type principal.

```swift
import AppKit

private func shortMenuTitle(_ title: String) -> String {
  if title.count <= 30 {
    return title
  }
  return String(title.prefix(27)) + "..."
}

final class AppDelegate: NSObject, NSApplicationDelegate {
  func applicationDidFinishLaunching(_ notification: Notification) {
    NSApp.setActivationPolicy(.regular)
    NSApp.activate(ignoringOtherApps: true)
  }
}

@main
struct SampleApp: App {
  @NSApplicationDelegateAdaptor(AppDelegate.self) private var appDelegate

  var body: some Scene {
    WindowGroup("Sample", id: "main") {
      ContentView()
    }

    MenuBarExtra("Sample", systemImage: "bolt.circle") {
      Button(shortMenuTitle("Open Dashboard")) { /* ouvrir la fenêtre */ }
      Divider()
      Button("Quit") {
        NSApplication.shared.terminate(nil)
      }
    }
  }
}
```

## Pièges

- Ne pas compter uniquement sur une scene `Window(...)` pour la fenêtre de lancement principale d'une app menu-bar-plus-window quand le produit attend une fenêtre normale au démarrage.
- Ne pas livrer silencieusement une app menu-bar-only sans Dock si l'utilisateur attend un process d'app normal. Soit installer l'app delegate et passer à `.regular`, soit documenter clairement que le comportement `.accessory` est intentionnel.
- Ne pas transformer le menu bar extra en un petit substitut surchargé pour une fenêtre d'app complète.
- Ne pas rendre des titres ou corps de message bruts non bornés comme menu items. Les labels longs font vite exploser la largeur du menu et devraient être plafonnés à 30 caractères avec un titre d'affichage court.
- En cas de besoin d'une customisation avancée du status item ou d'un contrôle de menu AppKit, utiliser la skill `erom-dev-macos-apps:appkit-interop`.
