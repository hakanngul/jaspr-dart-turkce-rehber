# 11. CSS ve UI Geliştirme

## 11.1 Üç Stil Yaklaşımı

Jaspr stil konusunda kasıtlı olarak esnektir: kendi tip-güvenli CSS-in-Dart sistemi, harici stil dosyaları ve CSS framework'leri birlikte kullanılabilir.

| Yaklaşım | Nasıl | Ne zaman |
|----------|-------|----------|
| Inline Styles sınıfı | `styles: Styles(...)` | Tek elemente özgü, küçük stiller |
| @css component stilleri | Component içinde `@css static List<StyleRule> get styles` | Önerilen varsayılan — stil component'le birlikte yaşar |
| Harici .css dosyası | `link(rel: 'stylesheet', href: '/styles.css')` | Büyük paylaşılan temalar, mevcut CSS'in taşınması |
| CSS framework | Tailwind (jaspr_tailwind), Bulma, Sass (sass_builder) | Hazır tasarım sistemi istediğinde |

## 11.2 Styles Sınıfı (Tip-Güvenli CSS)

`Styles`, yaygın CSS özelliklerini tiplenmiş parametrelerle sunar:

```dart
div(
  styles: const Styles(
    backgroundColor: Colors.red,
    padding: Padding.all(8.px),
    radius: BorderRadius.all(Radius.circular(4.px)),
  ),
  [],
)
// Üretilen HTML: <div style="background-color: red; padding: 8px; border-radius: 4px;">
```

Birimler uzantılarla yazılır: `100.px`, `10.rem`, `50.percent`. Henüz tiplenmemiş bir CSS özelliği için `raw` kaçış kapısı vardır:

```dart
const myStyle = Styles(
  color: Colors.red,
  raw: {'some_advanced_css_property': 'special_value'},
);
```

Stilleri birleştirme ve yeniden kullanma:

```dart
const kirmiziMetin = Styles(color: Colors.red);

const kirmiziMaviUstunde = Styles.combine([
  kirmiziMetin,
  Styles(backgroundColor: Colors.blue),
]);
```

> [!NOTE]
> **İpucu**
> Mümkün olan her yerde `const` Styles kullan — sitenin performansına ve boyutuna katkı sağlar. `jaspr_lints` paketi stil özelliklerini düzenli tutan bir lint kuralı içerir.

## 11.3 @css ile Component Seviyesinde Stil (Scoped Styling)

Önerilen yaklaşım: stilleri component'in içinde tanımla; `&` seçicisi üst sınıfa referans verir:

```dart
class App extends StatelessComponent {
  const App({super.key});

  @override
  Component build(BuildContext context) {
    return div(classes: 'main', [
      p([.text('Merhaba Dünya')]),
    ]);
  }

  @css
  static List<StyleRule> get styles => [
    css('.main', [
      css('&').styles(
        width: 100.px,
        padding: Padding.all(10.rem),
      ),
      css('p').styles(
        color: Colors.blue,
      ),
    ]),
  ];
}
```

Bu kurallar otomatik olarak `<head>` içindeki global stil dosyasına işlenir. `pubspec.yaml`'a `jaspr: { styles: standalone }` eklersen kurallar satır içi yerine ayrı bir `.css` dosyasına çıkarılır (tarayıcı önbelleği açısından avantajlı; client modunda varsayılan budur).

> [!WARNING]
> **Dikkat (standalone modu)**
> `standalone` seçildiğinde `@css` içeren kütüphaneler build sırasında Dart VM'de çalıştırılır; bu dosyalarda `dart:js_interop`/`package:web` import edemezsin.

## 11.4 Global Stiller

- `Document(styles: [...])` parametresiyle (server/static) kurallar `<head>`'e yazılır.
- Global değişkende `@css` anotasyonu.
- Herhangi bir yerde `Style` component'i ile elle `<style>` etiketi.
- Harici dosya: `Document(head: [link(rel: 'stylesheet', href: '/styles.css')])` (client modunda doğrudan `web/index.html` içine).

## 11.5 Responsive Tasarım: Media Query

```dart
@css
static List<StyleRule> get styles => [
  css('.main').styles(
    display: Display.flex,
    flexDirection: FlexDirection.row,   // geniş ekran: yan yana
  ),
  css.media(MediaQuery.screen(maxWidth: 600.px), [
    css('.main').styles(
      flexDirection: FlexDirection.column, // dar ekran: alt alta
    ),
  ]),
];
```

## 11.6 Flexbox ve Grid

Flutter'daki `Row`/`Column` yerine CSS flexbox kullanılır:

```dart
// Flutter'daki Row karşılığı:
div(
  styles: Styles(
    display: Display.flex,
    flexDirection: FlexDirection.row,
    justifyContent: JustifyContent.spaceBetween,
    alignItems: AlignItems.center,
    gap: Gap.row(16.px),
  ),
  [ /* çocuklar */ ],
)

// Grid örneği:
div(
  styles: Styles(
    display: Display.grid,
    gridTemplate: GridTemplate(columns: TrackSizes([TrackSize(1.fr), TrackSize(1.fr)])),
    gap: Gap.all(12.px),
  ),
  [ /* kartlar */ ],
)
```

## 11.7 Tailwind ve Diğer CSS Araçları

- **Tailwind:** Topluluk paketi `jaspr_tailwind` ile entegre edilir; utility sınıfları `classes: 'flex p-4'` şeklinde kullanılır.
- **Bulma:** Hazır component sınıflarını doğrudan kullanırsın; JasprPad'de örnek mevcut.
- **Sass:** `sass_builder` paketiyle `web/styles.scss` yazıp üretilen `styles.css`'i `link` ile dahil edersin.

Unutma: Jaspr siteleri sıradan web siteleridir — CSS ekosistemindeki neredeyse her şeyi kullanabilirsin.

> [!TIP]
> **Alıştırma 11**
> 1. Profil kartını `@css` ile stillendir: kart, gölge, hover efekti (raw ile `':hover'` seçicisi araştır).
> 2. Üç sütunlu responsive bir galeri yap: 900px altında 2, 600px altında 1 sütuna düşsün.
> 3. Projeye `jaspr_tailwind` ekle ve bir butonu yalnızca utility class'larla stillendir.

---

[⬅ 10. Component Mimarisi](10-component-mimarisi.md) | [Sonraki: 12. Asset Kullanımı ➡](12-asset-kullanimi.md)
