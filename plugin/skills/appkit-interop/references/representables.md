# Representables

## Objectif

Utiliser cette référence pour envelopper un contrôle ou un controller AppKit dans une app SwiftUI macOS.

## Choisir le type de wrapper

- Utiliser `NSViewRepresentable` pour un pont au niveau vue, comme `NSTextView`, `NSScrollView`, ou un contrôle AppKit personnalisé.
- Utiliser `NSViewControllerRepresentable` quand il faut le cycle de vie d'un controller, la coordination de delegate, ou la logique de présentation AppKit.

## Squelette

```swift
struct LegacyTextView: NSViewRepresentable {
  @Binding var text: String

  func makeCoordinator() -> Coordinator {
    Coordinator(text: $text)
  }

  func makeNSView(context: Context) -> NSScrollView {
    let scrollView = NSScrollView()
    let textView = NSTextView()
    textView.delegate = context.coordinator
    scrollView.documentView = textView
    return scrollView
  }

  func updateNSView(_ nsView: NSScrollView, context: Context) {
    guard let textView = nsView.documentView as? NSTextView else { return }
    if textView.string != text {
      textView.string = text
    }
  }

  final class Coordinator: NSObject, NSTextViewDelegate {
    @Binding var text: String

    init(text: Binding<String>) {
      _text = text
    }

    func textDidChange(_ notification: Notification) {
      guard let textView = notification.object as? NSTextView else { return }
      text = textView.string
    }
  }
}
```

## Pièges

- Éviter les boucles de mise à jour infinies en ne poussant l'état vers AppKit que quand les valeurs ont réellement changé.
- Garder les delegates et le câblage target-action dans le coordinator.
- Si le wrapper grossit jusqu'à devenir un écran complet, réévaluer la frontière.
