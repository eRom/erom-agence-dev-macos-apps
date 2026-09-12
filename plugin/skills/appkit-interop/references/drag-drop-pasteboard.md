# Drag, Drop et Pasteboard

## Objectif

Utiliser cette référence quand le drag/drop desktop ou le comportement du pasteboard dépasse ce que les modificateurs SwiftUI classiques couvrent confortablement.

## Cas d'usage adaptés

- Drag de file URLs
- Interopérabilité pasteboard avec d'autres apps macOS
- Previews de drag riches ou validation de drop spécifique à AppKit
- Vues AppKit historiques avec des types de drag personnalisés

## Patterns de base

- Commencer par les API SwiftUI de drag/drop quand elles couvrent déjà le cas d'usage.
- Basculer vers AppKit quand il faut `NSPasteboard`, des types de pasteboard personnalisés, ou les anciens flux de delegate AppKit.
- Garder la conversion de données à la frontière, plutôt que de laisser fuir les types AppKit dans toute la fonctionnalité.

## Pièges

- Ne pas faire basculer toute la liste ou le canvas vers AppKit pour une seule cible de drop.
- Garder les types de fichier et de pasteboard explicites et validés.
