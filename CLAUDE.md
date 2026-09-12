# Contrat du dépôt

Dépôt de développement du plugin Claude Code `erom-dev-macos-apps` : Concevoir, construire et livrer des apps macOS natives en SwiftUI.
Ce fichier dit comment travailler ici. Il décrit un contrat, pas un état atteint :
la section « État actuel » dit ce qui n'y est pas encore conforme.

## Structure

```
plugin/                          seul dossier distribué (source marketplace git-subdir)
  .claude-plugin/plugin.json     manifeste
  skills/<nom>/SKILL.md          une skill par dossier
  skills/<nom>/references/       matière longue, lue seulement quand la skill tourne
  agents/<nom>.md                subagents, découverts automatiquement
docs/                            specs, plans, recherches, revues
_memory_/                        connaissance de session persistée, indexée par le vault RAG
.claude/                         notes et settings locaux, jamais distribués
```

Pas d'étape de build : les skills sont du Markdown pur, écrit à la main
directement dans `plugin/`.

## Invariants

1. **Seul `plugin/` est distribué.** Tout ce qui est hors de ce dossier reste
   local ou sert le développement : notes, matière de travail, mémoire.
2. **Français.** Le plugin sert des projets français. Skills, exemples et
   sorties en français, y compris les descriptions de `SKILL.md`.
3. **Aucune capacité dupliquée.** Avant d'ajouter une skill, vérifier qu'aucun
   autre plugin eRom ne la porte déjà (`~/dev/erom-marketplace/.claude-plugin/marketplace.json`
   liste les plugins publiés et ce qu'ils couvrent). Deux plugins qui font la
   même chose, c'est un plugin de trop.
4. **La règle vient de ce qui a déjà tourné.** Chaque consigne d'une skill doit
   pouvoir se rattacher à un artefact réel : une sortie observée, un test qui
   passe, un incident daté. Le générique ne décrit aucun usage et ne sert de
   source à rien.
5. **Le manifeste ne déclare pas ses agents.** La clé `agents` absente vaut
   découverte automatique de `plugin/agents/`. Une liste explicite fige les
   chemins et fait mentir `claude plugin details`, qui affiche alors « Agents (0) ».
6. **Pas de dossier `commands/`.** Il est déprécié dans les plugins Claude Code
   (https://code.claude.com/docs/en/plugins.md, vérifié le 2026-09-12). Une
   commande est une skill avec `disable-model-invocation: true` et
   `argument-hint`, invoquée `/erom-dev-macos-apps:<nom>`.
7. **Jargon technique en anglais.** La prose est en français, mais les termes
   Apple et Swift (scene, toolbar, entitlements, notarization, Liquid Glass...)
   restent tels quels : c'est ce que l'utilisateur tape et ce que la doc Apple
   emploie.

## Origine

Les skills sont portées depuis le plugin `build-macos-apps` d'OpenAI
([github.com/openai/plugins](https://github.com/openai/plugins), licence MIT),
traduites en français et adaptées à Claude Code. Copie de travail de la source :
`~/dev/tmp-plugin/build-macos-apps/`. Le dépôt amont n'a pas de fichier LICENSE,
le MIT n'y est déclaré que dans `.codex-plugin/plugin.json` : la mention est
reprise en pied de `plugin/LICENSE`.

## Vérifier

```bash
claude --plugin-dir plugin plugin details erom-dev-macos-apps
```

Affiche l'inventaire réel des composants chargés et le coût token projeté.
C'est la seule preuve que le manifeste charge ce qu'on croit : un dossier
`skills/` ou `agents/` peuplé mais invisible ici n'est pas chargé.

## Publication

Publié dans `erom-marketplace` en 0.1.0 le 2026-09-12. La publication ne se fait pas à la main : la skill `release`
du plugin `erom-dev-plugin` la porte de bout en bout, depuis ce dépôt.

```
/erom-dev-plugin:release
```

Elle lit le nom et la version dans le manifeste, choisit le bump SemVer, commite
et pousse ce dépôt, puis met à jour `~/dev/erom-marketplace` (entrée du plugin,
metadata, README), et vérifie la CI. Toujours dans cet ordre : le plugin d'abord, 
la marketplace ensuite, parce que l'entrée pointe `ref: main` sur ce dépôt.

Sur une **première** publication, elle s'arrête et demande : l'entrée à créer
réclame une description, une source `git-subdir` et un choix de `strict`. C'est
le moment de les préparer, pas avant.

## État actuel - 2026-09-12

14 skills portées depuis OpenAI : 11 déclenchées par le modèle, 3 lancées au slash
(ex-commands). Aucune n'a encore tourné sur un projet eRom.

Œuvre servie : passer la PWA `erom-agence-control-plane` en app macOS native.
Chaque skill de ce plugin doit se justifier par ce chantier tant qu'aucun autre
projet réel ne l'a rejoint.

**Écart à l'invariant 4.** Le contenu porté vient de l'expérience d'OpenAI, pas
d'une sortie observée ici. Il fait foi jusqu'à ce que le control-plane le
contredise : toute consigne qui casse sur ce chantier se corrige avec l'incident
daté à l'appui.

| Élément | État |
|---|---|
| Première cible | `erom-agence-control-plane` (PWA) vers app macOS native |
| `plugin/skills/` | 14 skills, portées, jamais éprouvées |
| Image du README | `assets/erom-dev-macos-apps.png`, livrée le 2026-09-12 |
| Publication marketplace | 0.1.0, première publication le 2026-09-12 |
