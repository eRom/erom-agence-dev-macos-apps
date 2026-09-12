# Responder Chain et Menus

## Objectif

Utiliser cette référence quand la gestion des commandes dépend de la fenêtre active, du first responder, ou de la validation de menu AppKit.

## Patterns de base

- Commencer par les `commands`, `FocusedValue`, et l'état de scene focused de SwiftUI.
- Utiliser les hooks de responder chain AppKit seulement quand le routage ou la validation de commande dépend vraiment du système de responder sous-jacent.
- Garder les règles d'activation de menu proches de l'état dont elles dépendent.

## Cas d'usage adaptés pour AppKit

- Valider si un item de menu doit être activé
- Router des actions via le first responder courant
- S'intégrer aux comportements de document ou de texte AppKit existants

## Pièges

- Ne pas recréer une gestion de commandes globale à l'AppKit quand les focused values SwiftUI feraient l'affaire.
- Éviter d'éparpiller la logique de commande entre des closures SwiftUI et des selectors AppKit sans frontière claire.
