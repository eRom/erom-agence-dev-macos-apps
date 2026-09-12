# erom-dev-macos-apps

Plugin Claude Code. Concevoir, construire et livrer des apps macOS natives en SwiftUI.

## Installation

```
/plugin marketplace add eRom/erom-marketplace
/plugin install erom-dev-macos-apps@erom-marketplace
```

## Les skills

Les skills se déclenchent seules quand la situation les appelle. Trois sont des
commandes, lancées uniquement au slash.

| Skill | Invocation | Ce qu'elle fait |
|---|---|---|
| `build-run-debug` | auto | Build, lancement et debug d'une app macOS via un script `script/build_and_run.sh` unique |
| `swiftpm-macos` | auto | Build, run et test d'un package SwiftPM sans projet Xcode |
| `test-triage` | auto | Lance le plus petit périmètre de tests utile et classe les échecs |
| `signing-entitlements` | auto | Diagnostic codesign, entitlements, sandbox, hardened runtime et Gatekeeper |
| `packaging-notarization` | auto | Prépare l'archive, le bundle et la notarization pour la distribution |
| `telemetry` | auto | Instrumentation `Logger` légère et vérification des événements avec `log stream` |
| `swiftui-patterns` | auto | Scenes, commands, toolbars, settings, split views et inspectors natifs macOS |
| `view-refactor` | auto | Découpe les views SwiftUI trop grosses vers une structure desktop stable |
| `window-management` | auto | Chrome, drag regions, placement, restauration et style des windows |
| `liquid-glass` | auto | Adoption de Liquid Glass et retrait du chrome custom qui le contredit |
| `appkit-interop` | auto | Pont SwiftUI vers AppKit au plus étroit : representables, panels, responder chain |
| `build-and-run-macos-app` | `/erom-dev-macos-apps:build-and-run-macos-app` | Crée ou met à jour le script de lancement, puis build et lance |
| `fix-codesign-error` | `/erom-dev-macos-apps:fix-codesign-error` | Inspecte une erreur de signature et donne le chemin de correction minimal |
| `test-macos-app` | `/erom-dev-macos-apps:test-macos-app` | Lance les tests ciblés et explique les échecs par catégorie |

## Origine

Les skills sont portées depuis le plugin `build-macos-apps` d'OpenAI
([github.com/openai/plugins](https://github.com/openai/plugins), licence MIT),
traduites en français et adaptées à Claude Code.

## Licence

MIT, Romain Ecarnot. Les parties issues de `build-macos-apps` restent sous la
licence MIT d'OpenAI, voir `LICENSE`.
