# Script de build et run

Ceci est le contrat canonique de bootstrap pour la boucle de run locale
macOS de cette skill.

Quand un projet n'a pas encore de point d'entrée run macOS établi :

1. Créer un `script/build_and_run.sh` local au projet.
2. Le rendre exécutable.
3. L'utiliser comme point d'entrée unique kill + build + run.
4. Supporter les flags optionnels `--debug`, `--logs`, `--telemetry`, et `--verify`.

## `script/build_and_run.sh`

Utiliser un seul script spécifique au projet avec un petit sélecteur de mode
et un chemin par défaut sans flag qui se contente de tuer, builder, et
lancer. Garder le lancement direct de l'exécutable réservé aux vrais outils
en ligne de commande. Pour les apps AppKit/SwiftUI GUI en SwiftPM, mettre en
place un bundle `.app` local au projet et lancer ce bundle avec
`/usr/bin/open -n`.

### Exécutable CLI SwiftPM

Utiliser cette forme pour les vrais outils en ligne de commande :

```bash
#!/usr/bin/env bash
set -euo pipefail

MODE="${1:-run}"
APP_NAME="MyTool"

pkill -x "$APP_NAME" >/dev/null 2>&1 || true

swift build
APP_BINARY="$(swift build --show-bin-path)/$APP_NAME"

case "$MODE" in
  run)
    "$APP_BINARY"
    ;;
  --debug|debug)
    lldb -- "$APP_BINARY"
    ;;
  --logs|logs)
    "$APP_BINARY" &
    /usr/bin/log stream --info --style compact --predicate "process == \"$APP_NAME\""
    ;;
  --telemetry|telemetry)
    "$APP_BINARY" &
    /usr/bin/log stream --info --style compact --predicate "subsystem == \"com.example.MyTool\""
    ;;
  --verify|verify)
    "$APP_BINARY" &
    sleep 1
    pgrep -x "$APP_NAME" >/dev/null
    ;;
  *)
    echo "usage: $0 [run|--debug|--logs|--telemetry|--verify]" >&2
    exit 2
    ;;
esac
```

### App GUI AppKit/SwiftUI en SwiftPM

Utiliser cette forme pour les apps GUI SwiftPM afin qu'elles se lancent
comme une vraie app au premier plan, avec activation Dock et métadonnées de
bundle :

```bash
#!/usr/bin/env bash
set -euo pipefail

MODE="${1:-run}"
APP_NAME="MyApp"
BUNDLE_ID="com.example.MyApp"
MIN_SYSTEM_VERSION="14.0"

ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
DIST_DIR="$ROOT_DIR/dist"
APP_BUNDLE="$DIST_DIR/$APP_NAME.app"
APP_CONTENTS="$APP_BUNDLE/Contents"
APP_MACOS="$APP_CONTENTS/MacOS"
APP_BINARY="$APP_MACOS/$APP_NAME"
INFO_PLIST="$APP_CONTENTS/Info.plist"

pkill -x "$APP_NAME" >/dev/null 2>&1 || true

swift build
BUILD_BINARY="$(swift build --show-bin-path)/$APP_NAME"

rm -rf "$APP_BUNDLE"
mkdir -p "$APP_MACOS"
cp "$BUILD_BINARY" "$APP_BINARY"
chmod +x "$APP_BINARY"

cat >"$INFO_PLIST" <<PLIST
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>CFBundleExecutable</key>
  <string>$APP_NAME</string>
  <key>CFBundleIdentifier</key>
  <string>$BUNDLE_ID</string>
  <key>CFBundleName</key>
  <string>$APP_NAME</string>
  <key>CFBundlePackageType</key>
  <string>APPL</string>
  <key>LSMinimumSystemVersion</key>
  <string>$MIN_SYSTEM_VERSION</string>
  <key>NSPrincipalClass</key>
  <string>NSApplication</string>
</dict>
</plist>
PLIST

open_app() {
  /usr/bin/open -n "$APP_BUNDLE"
}

case "$MODE" in
  run)
    open_app
    ;;
  --debug|debug)
    lldb -- "$APP_BINARY"
    ;;
  --logs|logs)
    open_app
    /usr/bin/log stream --info --style compact --predicate "process == \"$APP_NAME\""
    ;;
  --telemetry|telemetry)
    open_app
    /usr/bin/log stream --info --style compact --predicate "subsystem == \"$BUNDLE_ID\""
    ;;
  --verify|verify)
    open_app
    sleep 1
    pgrep -x "$APP_NAME" >/dev/null
    ;;
  *)
    echo "usage: $0 [run|--debug|--logs|--telemetry|--verify]" >&2
    exit 2
    ;;
esac
```

Lancer un binaire GUI SwiftPM directement peut produire une absence d'icône
Dock, une absence d'activation au premier plan, et des avertissements de
bundle identifier manquant. Si le bundle `.app` s'ouvre mais que la fenêtre
principale ne passe toujours pas au premier plan, l'entrypoint de l'app a
peut-être besoin de `NSApp.setActivationPolicy(.regular)` et
`NSApp.activate(ignoringOtherApps: true)`.

Adapter l'étape de build pour les projets Xcode en remplaçant `swift build`
par `xcodebuild -project ...` ou `xcodebuild -workspace ...`, puis lancer le
binaire `.app` buildé depuis DerivedData ou un chemin de build déterministe
local au projet. Garder la même interface à un seul script et les mêmes
flags de mode.
