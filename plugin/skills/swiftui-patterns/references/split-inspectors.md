# Split Views and Inspectors

## Intention

Utiliser ceci quand l'app profite d'un layout sidebar-detail stable, d'un contenu supplémentaire optionnel ou d'un panel inspector.

## Patterns principaux

- Privilégier un state de sélection explicite plutôt qu'une navigation push-only.
- Démarrer avec `NavigationSplitView` quand le layout correspond au modèle mental du système.
- N'utiliser un split manuel que si un sizing inhabituel ou une colonne custom toujours visible est nécessaire.
- Utiliser `inspector(isPresented:)` pour des contrôles de detail légers qui complètent le content principal.

## Exemple : sidebar + detail

```swift
struct LibraryRootView: View {
  @State private var selection: Item.ID?
  @State private var showInspector = false

  var body: some View {
    NavigationSplitView {
      SidebarList(selection: $selection)
    } detail: {
      DetailView(selection: selection)
        .inspector(isPresented: $showInspector) {
          InspectorView(selection: selection)
        }
    }
  }
}
```

## Exemple : row de sidebar native

Privilégier la forme d'une row source-list native :

```swift
List(selection: $selection) {
  ForEach(items) { item in
    HStack(spacing: 10) {
      Image(systemName: item.systemImage)
        .foregroundStyle(.secondary)
        .frame(width: 16)

      VStack(alignment: .leading, spacing: 2) {
        Text(item.title)
          .lineLimit(1)

        if let detail = item.detail {
          Text(detail)
            .font(.caption)
            .foregroundStyle(.secondary)
            .lineLimit(1)
        }
      }
    }
    .tag(item.id)
  }
}
.listStyle(.sidebar)
```

Garder chaque row à une icône et une ou deux lignes de texte. Mettre les métadonnées
plus riches dans le detail ou l'inspector plutôt que dans chaque row de sidebar.

## Exemple : fonds de split-view

Laisser la sidebar et le container split garder les fonds système pendant que le
contenu detail possède ses propres surfaces custom :

```swift
NavigationSplitView {
  List(selection: $selection) {
    ForEach(items) { item in
      Label(item.title, systemImage: item.systemImage)
        .tag(item.id)
    }
  }
  .listStyle(.sidebar)
} detail: {
  ScrollView {
    VStack(alignment: .leading, spacing: 16) {
      DetailSummaryCard(item: selectedItem)
      DetailMetricsCard(item: selectedItem)
    }
    .padding()
  }
}
```

Éviter les remplissages opaques de sidebar et de root split-pane par défaut :

```swift
NavigationSplitView {
  List(items) { item in
    SidebarCardRow(item: item)
  }
  .listStyle(.sidebar)
  .background(Color(nsColor: .windowBackgroundColor))
} detail: {
  DetailView(item: selectedItem)
    .background(Color(.white))
}
```

## Pièges

- Éviter de remplacer tout le layout racine par des conditionnels top-level quand la sélection change.
- Éviter de cacher trop de detail derrière des sheets modales quand un inspector ou une colonne secondaire conviendrait mieux.
- Si le layout nécessite une délégation split view AppKit ou une coordination de fenêtre avancée, utiliser la skill `erom-dev-macos-apps:appkit-interop`.
