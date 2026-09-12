---
name: build-run-debug
description: Build, lance et débogue des apps macOS avec des workflows shell-first Xcode et Swift. À utiliser pour lancer une app ou diagnostiquer un échec de build, de démarrage ou d'exécution.
---

# Build, run et debug

## Démarrage rapide

Cette skill met en place un point d'entrée unique `script/build_and_run.sh`,
puis utilise ce script comme chemin par défaut de build et de run.
L'utilisateur peut le lancer lui-même depuis Claude Code en tapant
`! ./script/build_and_run.sh` (le préfixe `!` exécute la commande dans la
session).

Préférer les workflows shell-first :

- `./script/build_and_run.sh` comme point d'entrée unique kill + build + run une fois qu'il existe
- `xcodebuild` pour les workspaces ou projets Xcode
- `swift build` plus un lancement direct de l'exécutable dans ce script pour les vrais outils en ligne de commande SwiftPM
- `swift build` plus la mise en place d'un bundle `.app` local au projet et un lancement via `/usr/bin/open -n` pour les apps AppKit/SwiftUI GUI en SwiftPM
- des flags de script optionnels pour `lldb`, `log stream`, la vérification de telemetry, ou des contrôles de process post-lancement

Ne pas supposer de simulateurs, d'interaction tactile, ou d'outillage propre au mobile.

Si une surface MCP consciente d'Xcode est déjà disponible et que l'utilisateur
la veut explicitement, l'utiliser uniquement là où elle s'applique. Garder cet
usage étroit et honnête : la préférer pour la découverte, le logging ou le
support de debug orientés Xcode, et ne pas forcer des workflows spécifiques
au simulateur sur des tâches purement macOS.

## Procédure

1. Repérer la forme du projet.
   - Chercher `.xcworkspace`, `.xcodeproj`, et `Package.swift`.
   - Si plusieurs candidats existent, expliquer le choix par défaut et l'ambiguïté.

2. Résoudre la cible lançable et le nom du process.
   - Pour Xcode, lister les schemes et préférer le scheme qui produit l'app, sauf si l'utilisateur en nomme un autre.
   - Pour SwiftPM, identifier les executable products quand c'est possible.
   - Séparer la gestion du lancement SwiftPM par type de produit :
     - utiliser un lancement direct de l'exécutable seulement pour les vrais outils en ligne de commande,
     - utiliser un bundle `.app` local au projet et généré pour les apps AppKit/SwiftUI GUI.
   - Déterminer le nom de l'app/process à tuer avant de relancer.

3. Créer ou mettre à jour `script/build_and_run.sh`.
   - Rendre le script spécifique au projet et exécutable.
   - Il doit toujours :
     1. arrêter l'app/process en cours d'exécution si présent,
     2. builder la cible macOS,
     3. lancer l'app ou l'exécutable fraîchement buildé.
   - Ajouter des flags optionnels pour le debug/l'inspection de logs :
     - `--debug` pour lancer sous `lldb` ou attacher le debugger
     - `--logs` pour streamer les logs du process après lancement
     - `--telemetry` pour streamer les logs unifiés filtrés sur le subsystem/category de l'app
     - `--verify` pour lancer l'app et confirmer que le process existe avec `pgrep -x <AppName>`
   - Garder le chemin par défaut sans flag simple : kill, build, run.
   - Préférer écrire un seul script qui possède ce workflow plutôt que de redemander sans cesse un `swift build` manuel, la localisation de l'artefact, puis une commande de run ad hoc.
   - Pour les apps GUI SwiftPM, faire builder le produit par le script, créer `dist/<AppName>.app`, copier le binaire vers `Contents/MacOS/<AppName>`, générer un `Contents/Info.plist` minimal avec `CFBundlePackageType=APPL`, `CFBundleExecutable`, `CFBundleIdentifier`, `CFBundleName`, `LSMinimumSystemVersion`, et `NSPrincipalClass=NSApplication`, puis lancer avec `/usr/bin/open -n <bundle>`.
   - Pour `--logs` et `--telemetry` sur une app GUI SwiftPM, lancer le bundle avec `/usr/bin/open -n` d'abord, puis streamer les logs unifiés avec `/usr/bin/log stream --info ...`.
   - Ne pas recommander de lancement direct de l'exécutable SwiftPM pour les apps AppKit/SwiftUI GUI.
   - Utiliser `references/build-and-run-script.md` comme source canonique pour
     la forme du script. Ne pas créer un second extrait faisant autorité dans
     une autre skill.
   - Garder le script de run en dehors du code source de l'app. Il appartient à `script/build_and_run.sh`, pas à `App/`, `Views/`, `Models/`, `Stores/`, `Services/`, ou `Support/`.

4. Builder et lancer via le script.
   - Par défaut, utiliser `./script/build_and_run.sh`.
   - Utiliser `./script/build_and_run.sh --debug`, `--logs`, `--telemetry`, ou `--verify` quand l'utilisateur demande un support debugger/logs/telemetry/vérification de process.

5. Résumer les échecs correctement.
   - Classer le blocage comme compiler, linker, signing, build settings, SDK/toolchain manquant, bug de script, ou lancement runtime.
   - Citer le plus petit extrait d'erreur utile et expliquer ce qu'il signifie.

6. Déboguer de la bonne façon.
   - Utiliser le mode `--logs` ou `--telemetry` du script pour la vérification de config, d'entitlement, de sandbox, et d'action-event.
   - Pour les apps GUI SwiftPM, si le bundle `.app` se lance mais que sa fenêtre ne passe toujours pas au premier plan, vérifier si l'entrypoint a besoin de `NSApp.setActivationPolicy(.regular)` et `NSApp.activate(ignoringOtherApps: true)`.
   - Utiliser le mode `--debug` du script ou `lldb` directement si un debug de crash symbolisé est nécessaire.
   - Si l'utilisateur a besoin d'instrumenter et de vérifier des actions précises de window, sidebar, menu, ou menu bar, basculer sur la skill `erom-dev-macos-apps:telemetry`.
   - Garder des preuves resserrées et lisibles pour l'utilisateur.

7. Utiliser l'outillage MCP conscient d'Xcode seulement quand ça aide.
   - Si l'utilisateur demande explicitement XcodeBuildMCP et qu'il est déjà disponible, le préférer à un setup ad hoc.
   - Utiliser le MCP pour la découverte consciente d'Xcode ou les workflows de debug/logging quand la surface d'outils disponible correspond clairement à la tâche.
   - Revenir immédiatement aux commandes shell quand le MCP ne fournit pas un chemin macOS propre.

## Commandes de référence

- Découverte de projet :
  - `find . -name '*.xcworkspace' -o -name '*.xcodeproj' -o -name 'Package.swift'`
- Découverte de scheme :
  - `xcodebuild -list -workspace <workspace>`
  - `xcodebuild -list -project <project>`
- Build/run :
  - `./script/build_and_run.sh`
  - `./script/build_and_run.sh --debug`
  - `./script/build_and_run.sh --logs`
  - `./script/build_and_run.sh --telemetry`
  - `./script/build_and_run.sh --verify`

## Références

- `references/build-and-run-script.md` : source canonique de la forme du script `build_and_run.sh`.

## Garde-fous

- Préférer la commande la plus étroite qui prouve ou infirme la théorie en cours.
- Ne pas laisser l'utilisateur avec une chaîne de commandes manuelles ponctuelle une fois qu'un script `build_and_run.sh` stable peut posséder le workflow.
- Ne pas lancer une app GUI SwiftUI/AppKit en SwiftPM comme un exécutable brut sauf si l'utilisateur veut explicitement diagnostiquer ce mode d'échec : ça peut produire une absence d'icône Dock, une absence d'activation au premier plan, et des avertissements de bundle identifier manquant. Garder le lancement direct de l'exécutable réservé aux vrais outils en ligne de commande.
- Ne pas affirmer un état d'UI qu'on ne peut pas inspecter directement.
- Ne pas décrire des workflows mobile ou simulateur comme s'ils s'appliquaient à macOS.
- Si la sortie de build est énorme, résumer le premier vrai blocage et pointer vers des commandes de suivi.

## Sortie attendue

Fournir :
- le type de projet détecté
- le chemin du script configuré
- la commande exécutée
- si le build et le lancement ont réussi
- le blocage principal en cas d'échec
- la plus petite action suivante sensée
