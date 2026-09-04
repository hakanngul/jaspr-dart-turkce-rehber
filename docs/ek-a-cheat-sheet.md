# Ek A. Cheat Sheet — En Sık Kullanılan Yapılar

## Komutlar

```bash
dart install jaspr_cli        # CLI kurulumu (güncel, önerilen)
jaspr create proje_adi        # yeni proje
jaspr serve                   # geliştirme sunucusu (hot reload)
jaspr build                   # production build → build/jaspr
jaspr build --sitemap-domain https://site.com   # sitemap'li build
jaspr migrate                 # eski sürümden otomatik geçiş
jaspr install-skills          # AI araçları için Jaspr skill'leri
dart test                     # testleri çalıştır
```

## Component Kalıpları

```dart
// StatelessComponent
class X extends StatelessComponent {
  const X({super.key});
  @override
  Component build(BuildContext context) => div([.text('Merhaba')]);
}

// StatefulComponent
class Y extends StatefulComponent {
  const Y({super.key});
  @override
  State<Y> createState() => _YState();
}
class _YState extends State<Y> {
  @override
  Component build(BuildContext context) => div([]);
}

// HTML component'i
div(id: 'a', classes: 'x y', styles: Styles(...),
    attributes: {'data-k': 'v'}, events: {'click': (e) {}},
    [ .text('çocuk') ])

// Temel fabrikalar
.text('metin')  .fragment([...])  Component.empty()  RawText('<b>ham</b>')
```

## Routing

```dart
Router(routes: [
  Route(path: '/', builder: (c, s) => Home()),
  Route(path: '/u/:id', builder: (c, s) => User(id: s.params['id']!)),
  ShellRoute(builder: (c, s, child) => Yerlesim(child: child), routes: [...]),
  Route.lazy(path: '/a', load: a.loadLibrary, builder: (c, s) => a.A()),
])
Link(href: '/about', [.text('Git')])
context.push('/yol'); context.back(); context.pushNamed('ad', params: {...});
```

## Stil

```dart
Styles(color: Colors.blue, padding: Padding.all(8.px), raw: {...})
@css static List<StyleRule> get styles => [ css('.x').styles(...) ];
css.media(MediaQuery.screen(maxWidth: 600.px), [ ... ])
```

## Sunucu / İstemci

```dart
@client class X ...           // istemcide hydrate edilir
@sync int deger = 0;          // sunucu→istemci state (SyncMixin gerekli)
PreloadStateMixin             // sunucuda async ön-yükleme
AsyncBuilder(builder: (c) async => ...)   // yalnızca sunucu
context.url / context.headers / context.cookies / context.setStatusCode(404)
kIsWeb                        // istemcide miyim?
Document(title: ..., meta: {...})          // server/static kök
Document.head(title: ..., meta: {...})     // herhangi bir yerden head kontrolü
```

## Event'ler

```dart
button(onClick: () {}, [...])
input(type: InputType.text, onInput: (String v) {}, [])
events: events(onClick: () {}, onInput: (String v) {})
events: {'submit': (e) => e.preventDefault()}
```

---

[⬅ 18. Gerçek Projeler](18-gercek-projeler.md) | [Sonraki: Ek B — Sık Yapılan Hatalar ➡](ek-b-sik-hatalar.md)
