---
name: telemetry
description: Ajoute et vérifie une télémétrie runtime macOS légère. À utiliser pour câbler des événements Logger ou inspecter des logs de fenêtres, sidebars, menus et actions.
---

# Télémétrie

## Démarrage rapide

Utiliser cette skill pour ajouter une instrumentation d'app légère qui aide à
déboguer le comportement sans transformer le codebase en décharge de logs.
Préférer les API de unified logging d'Apple et vérifier les événements après
une boucle build/run.

## Principes

- Préférer `Logger` du framework `OSLog` pour les logs d'app structurés.
- Donner à chaque feature une paire subsystem/category claire pour que le filtrage runtime reste simple.
- Logguer les événements significatifs de cycle de vie utilisateur et app : ouverture de window, changements de sélection de sidebar, commandes de menu, actions de menu bar extra, jalons de sync/load, et chemins de fallback inattendus.
- Garder les logs info concis et stables. Utiliser les logs debug pour les détails d'état bruyants.
- Ne jamais logger de secrets, de tokens d'authentification, de données personnelles ou le contenu brut de documents.
- Ajouter des signposts seulement pour mesurer des durées ou des spans de performance ; ne pas sur-instrumenter par défaut.

## Pattern Logger minimal

```swift
import OSLog

private let logger = Logger(
  subsystem: Bundle.main.bundleIdentifier ?? "SampleApp",
  category: "Sidebar"
)

@MainActor
func selectItem(_ item: SidebarItem) {
  logger.info("Selected sidebar item: \(item.id, privacy: .public)")
  selection = item.id
}
```

Utiliser des categories spécifiques à la feature comme `Windowing`, `Commands`,
`MenuBar`, `Sidebar`, `Sync` ou `Import` pour que les logs se filtrent rapidement.

## Procédure

1. Identifier le comportement qui a besoin d'observabilité.
   - Ouverture/fermeture de window
   - Changements de sélection de sidebar ou d'inspector
   - Actions de commande menu ou clavier
   - Actions de menu bar extra
   - Événements de load/sync/import en arrière-plan
   - Chemins d'erreur et de récupération

2. Ajouter la plus petite instrumentation utile.
   - Créer un `Logger` par zone de feature ou par type.
   - Logguer les limites d'action et les transitions d'état clés.
   - Préférer une ligne à fort signal par action utilisateur plutôt que des dumps de valeurs bruyants.

3. Build et run de l'app.
   - Utiliser la skill `erom-dev-macos-apps:build-run-debug` pour la boucle build/run.
   - Si `script/build_and_run.sh` existe, préférer `./script/build_and_run.sh --telemetry` pour des vérifications de télémétrie en direct, ou `./script/build_and_run.sh --logs` pour des logs de processus plus larges.
   - Exercer le chemin UI ou commande censé émettre la télémétrie.

4. Lire les logs runtime et vérifier que l'événement s'est déclenché.
   - Utiliser Console.app avec un filtre process/subsystem quand c'est la vérification manuelle la plus rapide.
   - Utiliser `log stream --style compact --predicate 'process == "AppName"'` pour une vérification terminal en direct.
   - Préférer des prédicats plus resserrés quand le subsystem/category est connu :
     `log stream --style compact --predicate 'subsystem == "com.example.app" && category == "Sidebar"'`

5. Resserrer ou retirer l'instrumentation.
   - Si l'événement se déclenche, ne garder que les logs qui restent utiles pour un débogage futur.
   - S'il ne se déclenche pas, rapprocher le log du chemin de contrôle suspecté et relancer.

## Checklist de vérification

- L'app build après les changements de télémétrie.
- L'action concernée émet exactement une ligne de log claire, ou une petite séquence bornée.
- Le log peut être filtré par process, subsystem ou category.
- Aucun payload sensible n'est écrit dans les unified logs.
- Les logs de debug temporaires et bruyants sont retirés ou rétrogradés avant de finir.

## Garde-fous

- Ne pas utiliser `print` comme mécanisme principal de télémétrie d'app pour du code macOS.
- Ne pas laisser une trace dense de logs debug permanents autour de chaque mutation d'état.
- Ne pas affirmer qu'un événement est câblé correctement avant d'avoir un chemin de vérification concret via Console, `log stream`, ou une capture de sortie de processus.
- Si la tâche de débogage porte surtout sur l'analyse de crash/backtrace plutôt que sur la télémétrie d'action, passer à la skill `erom-dev-macos-apps:build-run-debug`.
