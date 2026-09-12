# Architecture

Mise à jour : 2026-09-12

## Type et objectif

Plugin Claude Code `erom-dev-macos-apps`, publié dans `erom-marketplace` en 0.1.0 le 2026-09-12.
Il aide à concevoir, builder, tester, signer et distribuer une app macOS native en SwiftUI.

Œuvre servie : passer la PWA `erom-agence-control-plane` en app macOS native. Aucune skill
n'a encore tourné sur ce chantier au 2026-09-12.

## Origine

Port du plugin Codex `build-macos-apps` d'OpenAI (github.com/openai/plugins, MIT déclaré
dans leur manifeste seulement). Skills traduites en français, adaptées à Claude Code.
Non repris : `assets/`, `agents/`, README d'origine, `agents/openai.yaml` de chaque skill.
Copie de travail de la source : `~/dev/tmp-plugin/build-macos-apps/`.

## Stack

Markdown pur, aucune étape de build. Pas de serveur MCP, pas d'agent, pas de hook.
Les skills pilotent des outils de la machine de l'utilisateur : `xcodebuild`, `swift`,
`open`, `lldb`, `codesign`, `spctl`, `plutil`, `log stream`.

## Arborescence

```
plugin/                          seul dossier distribué (git-subdir, ref main, strict true)
  .claude-plugin/plugin.json     manifeste, description longue = inventaire réel
  skills/<nom>/SKILL.md          14 skills
  skills/<nom>/references/       matière longue (swiftui-patterns, appkit-interop,
                                 window-management, build-run-debug)
  README.md, LICENSE             LICENSE porte la mention d'origine OpenAI en pied
assets/erom-dev-macos-apps.png   carte du README, 1536x1024, fusain
docs/                            vide
_memory_/                        cartographie de session (ONBOARD.md gitignoré)
```

## Composants

- 11 skills déclenchées par le modèle, par famille :
  - UI : `swiftui-patterns`, `window-management`, `liquid-glass`, `view-refactor`, `appkit-interop`
  - build et test : `build-run-debug`, `swiftpm-macos`, `test-triage`, `telemetry`
  - distribution : `signing-entitlements`, `packaging-notarization`
- 3 skills au slash uniquement (ex-commands Codex) : `build-and-run-macos-app`,
  `test-macos-app`, `fix-codesign-error`. Chacune renvoie à sa skill soeur
  (`build-run-debug`, `test-triage`, `signing-entitlements`) chargée par l'outil Skill.

## Flux

Point d'entrée unique de lancement dans les projets cibles : `script/build_and_run.sh`
(modes `--debug`, `--logs`, `--telemetry`, `--verify`), défini une seule fois dans
`plugin/skills/build-run-debug/references/build-and-run-script.md`. L'utilisateur le lance
lui-même avec `! ./script/build_and_run.sh`.

Coût mesuré par `claude plugin details` : ~1 300 tokens always-on, 0,5k à 5,2k par skill à l'invocation.
