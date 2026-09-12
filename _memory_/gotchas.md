# Pièges

Mise à jour : 2026-09-12

## Claude Code

- `commands/` est déprécié dans les plugins (https://code.claude.com/docs/en/plugins.md, lu le 2026-09-12). Une commande se porte en skill `disable-model-invocation: true` + `argument-hint`. `$ARGUMENTS`, `${CLAUDE_PLUGIN_ROOT}` et `${CLAUDE_SKILL_DIR}` sont substitués dans le corps d'une SKILL.md.
- `claude plugin details` affiche un coût always-on (~80 à 110 tokens) même pour les skills `disable-model-invocation`. [candidat 1x - plugin details du 2026-09-12]

## Agents traducteurs

- Un Write de Sonnet laisse des balises d'appel d'outil en fin de fichier (`</content>`, parfois `</invoke>`) : 5 fichiers du lot B, plus 1 corrigé par l'agent du lot A lui-même, le 2026-09-12. Contrôle : `command grep -rnE '</?(content|invoke|parameter)' plugin/` doit renvoyer zéro ligne. Imposer Edit plutôt que Write aux agents de retouche.
- Deux agents en parallèle sur le même brief ont pris deux registres (tutoiement contre infinitif). Le brief doit fixer le registre explicitement. [candidat 1x - port du 2026-09-12]
- Une consigne « tape `! ...` » écrite dans une skill s'adresse à tort à Claude : c'est un geste de l'utilisateur, écrire « L'utilisateur peut le lancer en tapant ». Corrigé dans `build-run-debug` et `swiftui-patterns`.

## Carte (illustrate)

- GPT Image a raté « LIVRER » deux fois de suite (« LIVER » au brouillon, « LIVERR » au tirage high). Corrigé par une édition qui remplace le mot entier (« SIGNER ET DISTRIBUER ») : l'édition de mot entier a tenu, le reste de la planche est resté identique au crop. [candidat 1x - carte du 2026-09-12]
- Le sous-agent a annoncé `erom-dev-macos-apps.png` alors que le fichier était sorti sans extension : lister `assets/` avant de lire.

## Contenu hérité d'OpenAI, non vérifié

- `liquid-glass` suppose les API de macOS 26 sans jamais nommer la version minimale.
- `menu-bar-extra.md` impose 30 caractères max pour un libellé de menu, sans source Apple.
- `swiftui-patterns` et `view-refactor` : un seul fichier Swift seulement sous ~50 lignes, heuristique non justifiée.
- `windowing.md` : une scene `Window(...)` pourrait ne pas s'ouvrir au lancement dans une app menu bar.

À trancher sur le chantier control-plane, avec l'incident daté à l'appui.

## Licence

- Le dépôt `openai/plugins` n'a aucun fichier LICENSE (vérifié par `gh api repos/openai/plugins/license`, 404) : le MIT n'est déclaré que dans `.codex-plugin/plugin.json`.
