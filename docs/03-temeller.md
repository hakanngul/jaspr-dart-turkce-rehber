# 3. Jaspr Temelleri: Component Sistemi ve HTML

## 3.1 Component Mantığı

Jaspr'da UI'ın her parçası bir **Component**'tir. Flutter'daki "Widget" kelimesinin yerini "Component" almıştır; yapı ve davranış büyük ölçüde aynıdır. Üç temel component tipi vardır: `StatelessComponent`, `StatefulComponent` ve `InheritedComponent`.

### StatelessComponent — Durumsuz Component

```dart
import 'package:jaspr/jaspr.dart';

class SelamComponent extends StatelessComponent {
  const SelamComponent({super.key});

  @override
  Component build(BuildContext context) {
    return p([.text('Merhaba Dünya')]);
  }
}
```

Satır satır:

- `extends StatelessComponent`: Flutter'daki `StatelessWidget`'ın karşılığı.
- `const SelamComponent({super.key})`: Key parametresi alan const constructor — Flutter'dakiyle aynı.
- `build(BuildContext context)`: Ağaçtaki konum bilgisini taşıyan context'i alır, tek bir `Component` döndürür.
- `p([...])`: HTML `<p>` etiketi üreten component. Son parametre olarak çocuk listesi alır.
- `.text('...')`: Dart 3.10 dot-shorthand yazımıyla `Component.text('...')` — HTML etiketi olmayan düz metin düğümü üretir.

### StatefulComponent — Durumlu Component

```dart
class Sayac extends StatefulComponent {
  const Sayac({super.key});

  @override
  State<Sayac> createState() => SayacState();
}

class SayacState extends State<Sayac> {
  int sayi = 0;

  @override
  Component build(BuildContext context) {
    return div([
      p([.text('Sayaç: $sayi')]),
      button(
        onClick: () => setState(() => sayi++),
        [.text('Artır')],
      ),
    ]);
  }
}
```

Flutter'dan farkı yok denecek kadar azdır: `createState()`, `State<T>` sınıfı, `setState()` aynen çalışır. `setState` çağrıldığında Jaspr yalnızca etkilenen DOM düğümlerini günceller.

### InheritedComponent — Ağaçta Veri Paylaşımı

Flutter'daki `InheritedWidget`'ın birebir karşılığıdır; veriyi ağacın altındaki component'lere verimli şekilde ulaştırır:

```dart
class TemaComponent extends InheritedComponent {
  const TemaComponent({required super.child, super.key});

  static TemaComponent of(BuildContext context) {
    final result = context
        .dependOnInheritedComponentOfExactType<TemaComponent>();
    assert(result != null, 'Ağaçta TemaComponent bulunamadı');
    return result!;
  }

  @override
  bool updateShouldNotify(covariant TemaComponent oldComponent) {
    return false; // veri değiştiğinde true döndür
  }
}
```

## 3.2 HTML Elementleri Oluşturma

Jaspr'da her yaygın HTML etiketi bir component fonksiyonu olarak tanımlıdır ve `package:jaspr/dom.dart` kütüphanesinden gelir. Şu HTML'i düşün:

```html
<div>
  <h1>Bu bir başlık</h1>
  <p>Merhaba <b>Dünya!</b></p>
</div>
```

Jaspr karşılığı:

```dart
div([
  h1([.text('Bu bir başlık')]),
  p([.text('Merhaba '), b([.text('Dünya!')])]),
]);
```

`div`, `a`, `p`, `img` gibi yaygın etiketlerin yanında `video`, `form`, `input`, `select` gibi özel etiketler de hazır gelir. Bazı component'ler etikete özgü parametreler alır: `img(src: "...")`, `a(href: "...")`, `input(type: InputType.text)` gibi.

### Örnekler

```dart
// Zengin metin paragrafı
p([.text('Bu biraz '), b([.text('kalın')]), .text(' içerik.')])

// Mavi başlık
h1(styles: Styles(color: Colors.blue), [.text('Merhaba Jaspr!')])

// İçinde resim olan bağlantı
a(href: 'https://github.com/schultek/jaspr', target: .blank, [
  img(src: '/images/logo.png'),
])

// Açılır liste
select([
  option(value: 'a', [.text('Beni seç!')]),
  option(value: 'b', selected: true, [.text('Ya da beni!')]),
])

// İlerleme çubuğu
progress(value: 85, max: 100, [])
```

> [!NOTE]
> **Okunabilirlik Kuralı**
> Çocuk listesini parametre listesinin en sonuna koy: `div(id: 'main', [.text('Merhaba')])` yazımı, `div([...], id: 'main')` yazımından daha okunaklıdır. Çocuk yoksa boş liste ver: `[]`.

## 3.3 Attribute, Class ve Style Kullanımı

Her HTML component'i şu ortak parametreleri alır:

```dart
div(
  id: 'ana-kutu',                        // id attribute
  classes: 'kart golgeli',               // class attribute (boşlukla ayrılmış)
  styles: Styles(color: Colors.black),   // inline style
  attributes: {'data-izleme': '123'},    // isteğe bağlı attribute'lar
  events: {'click': (e) => print('tık')}, // DOM event'leri
  key: myKey,                            // component anahtarı
  [ /* çocuklar */ ],
);
```

Üretilen HTML:

```html
<div id="ana-kutu" class="kart golgeli" style="color: black;" data-izleme="123">...</div>
```

## 3.4 Temel Component Fabrikaları

Flutter'dan farklı olarak Jaspr'da sınırlı sayıda temel (foundational) component vardır; çünkü layout ve boyama işini tarayıcı yapar. Hepsi `Component` sınıfı üzerindeki fabrika constructor'larıdır:

| Fabrika | Görevi |
|---------|--------|
| `Component.element(tag: ...)` | Etiket adını string olarak verdiğin alt seviye HTML elementi. Genelde hazır `div()` vb. tercih edilir. |
| `Component.text('...')` / `.text('...')` | Düz metin düğümü. |
| `Component.fragment([...])` / `.fragment([...])` | Sarmalayıcı etiket olmadan birden çok çocuk render eder. `build()` tek component döndürmek zorunda olduğunda çok işe yarar. |
| `Component.empty()` | Hiçbir şey render etmez; "boş dön" ihtiyacında kullanılır. |
| `Component.wrapElement(...)` | Kendi etiket üretmeden attribute/style'ları doğrudan çocuğun elementine uygular. |

```dart
// Fragment örneği — tek component döndürme zorunluluğunda:
@override
Component build(BuildContext context) {
  return .fragment([
    h1([.text('Hoş geldin')]),
    p([.text('Merhaba Dünya')]),
  ]);
}
// Üretilen HTML: <h1>Hoş geldin</h1><p>Merhaba Dünya</p>
```

> [!WARNING]
> **Eski Kullanım / Güncel Kullanım (0.22)**
> Eski: `text('Merhaba')`, `fragment([...])`, `raw('<div>...</div>')` fonksiyonları.
> Güncel: `Component.text('Merhaba')` / `.text('Merhaba')`, `Component.fragment([...])` / `.fragment([...])`, `RawText('...')`. Eski fonksiyonlar deprecated'tir; `jaspr migrate` otomatik dönüştürür. HTML component'leri 0.22'de fonksiyondan sınıfa çevrildi — artık `const div([...])` yazabilirsin; bu performans için önerilir.

## 3.5 Component Composition (Bileşim)

UI'ı küçük, parametreli component'lere bölüp birleştirmek temel çalışma biçimidir:

```dart
class Kart extends StatelessComponent {
  const Kart({required this.baslik, required this.icerik, super.key});

  final String baslik;
  final String icerik;

  @override
  Component build(BuildContext context) {
    return div(classes: 'kart', [
      h3([.text(baslik)]),
      p([.text(icerik)]),
    ]);
  }
}

// Kullanım:
div([
  Kart(baslik: 'Birinci', icerik: 'İlk kart içeriği.'),
  Kart(baslik: 'İkinci', icerik: 'İkinci kart içeriği.'),
])
```

Küçük component'ler hem okunabilirliği artırır hem de test edilebilirliği kolaylaştırır. Büyük projelerde bu yaklaşımın nasıl organize edileceği Bölüm 10'dadır.

## 3.6 Yaygın Hatalar

> [!WARNING]
> **Hata 1: build() içinde liste döndürmeye çalışmak**
> `build()` tek bir `Component` döndürür. Birden çok kök element gerekiyorsa `.fragment([...])` kullan.

> [!WARNING]
> **Hata 2: Metni doğrudan children listesine koymak**
> `div(['Merhaba'])` derlenmez; metin her zaman `.text('Merhaba')` ile sarılmalıdır.

> [!WARNING]
> **Hata 3: Sunucu component'inde web kütüphanesi import etmek**
> Sadece sunucuda render edilen bir component'te `package:web` veya `dart:js_interop` import edersen derleme hatası alırsın. Çözüm Bölüm 9 ve 13'te.

> [!TIP]
> **Alıştırma 3**
> 1. Kendi adını, kısa bir biyografini ve sosyal medya bağlantılarını gösteren bir "Profil Kartı" component'i yaz; sayfada iki farklı kişi için kullan.
> 2. Sayaç component'ine "Azalt" ve "Sıfırla" butonları ekle.
> 3. `.fragment()` kullanarak başlık + paragraf + resim içeren bir component yaz; üretilen HTML'i tarayıcıda incele.

---

[⬅ 2. Kurulum](02-kurulum.md) | [Sonraki: 4. Dart ile Jaspr Geliştirme ➡](04-dart-ile-gelistirme.md)
