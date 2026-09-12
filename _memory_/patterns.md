# Patterns et conventions

Mise à jour : 2026-09-12

## Langue des skills

- Prose en français, consignes à l'infinitif (« Utiliser X », « Ne pas coder en dur »), jamais de tutoiement. Intro de skill : « Utiliser cette skill pour... » ou « Cette skill ... ».
- Jargon Apple et Swift en anglais : scene, window, toolbar, sidebar, inspector, entitlements, hardened runtime, notarization, Liquid Glass, build (verbe conjugué « builder »).
- Code intact ; seuls les commentaires des blocs de code passent en français. Chaînes UI des exemples (`Button("New Note")`) laissées telles quelles.
- Aucun tiret cadratin ni demi-cadratin dans `plugin/` et les README (contenu public).

## Frontmatter

- Skill modèle : `name` (anglais, = dossier) + `description` française « ce que fait la skill. À utiliser quand ... ».
- Skill au slash : `description` finissant par « Commande explicite, lancée au slash : /erom-dev-macos-apps:<nom>. », `argument-hint` en `[clé=valeur]`, `user-invocable: true`, `disable-model-invocation: true`, et `Arguments reçus : $ARGUMENTS` dans le corps.

## Renvois

- Skill soeur : `erom-dev-macos-apps:<skill>`.
- Référence de la même skill : chemin relatif `references/<fichier>.md`.
- Référence d'une autre skill depuis une skill au slash : `${CLAUDE_PLUGIN_ROOT}/skills/<skill>/references/<fichier>.md`.

## Commits et release

- Style : `feat: ...`, `chore(release): ... (<version>)`, corps en français, trailers `Co-Authored-By` et `Claude-Session`.
- Release par `/erom-dev-plugin:release` : plugin poussé d'abord, marketplace ensuite. Nouveau plugin = metadata marketplace en mineur (0.27.0 -> 0.28.0), comme `erom-vision` et `erom-dev-ios-apps`.
- Vérification de chargement : `claude --plugin-dir plugin plugin details erom-dev-macos-apps` doit lister 14 skills.

## Port d'un plugin tiers

Brief commun dans un fichier, lots parallèles par skill sous Sonnet, puis relecture par le pilote : longueurs, titres, puces et blocs de code comparés à la source, grep final (tirets, `codex|openai`, balises parasites, tutoiement).
