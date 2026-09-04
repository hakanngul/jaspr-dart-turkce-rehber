# 6. State Management (Durum Yönetimi)

## 6.1 Jaspr'da State Nasıl Çalışır?

Jaspr'ın state modeli Flutter ile aynıdır: component ağacı, `setState()` ile tetiklenen yeniden build'ler ve veriyi ağaçta aşağı taşıyan `InheritedComponent`'ler. Fark, render hedefinin DOM olmasıdır — `setState` çağrıldığında Jaspr yalnızca değişen DOM düğümlerini günceller.

## 6.2 Component State (Yerel Durum)

Durum yalnızca tek component'i ilgilendiriyorsa `StatefulComponent` + `setState` yeterlidir:

```dart
class AcilirMenu extends StatefulComponent {
  const AcilirMenu({super.key});
  @override
  State<AcilirMenu> createState() => _AcilirMenuState();
}

class _AcilirMenuState extends State<AcilirMenu> {
  bool acik = false;

  @override
  Component build(BuildContext context) {
    return div([
      button(
        onClick: () => setState(() => acik = !acik),
        [.text(acik ? 'Kapat' : 'Aç')],
      ),
      if (acik) ul([
        li([.text('Seçenek 1')]),
        li([.text('Seçenek 2')]),
      ]),
    ]);
  }
}
```

Component'in bilinen yaşam döngüsü metodları burada da geçerlidir: `initState()`, `didChangeDependencies()`, `dispose()`. `initState`'te abonelikleri başlat, `dispose`'da iptal et.

## 6.3 Global State Yaklaşımları

### Yaklaşım 1: InheritedComponent (kurumsuz)

Birkaç component arasında paylaşılan basit veri için yeterlidir (Bölüm 3.1'deki örnek). Elle yazımı biraz uzundur ama ek bağımlılık gerektirmez.

### Yaklaşım 2: jaspr_riverpod (önerilen)

Jaspr'ın component sistemi Flutter'a çok yakın olduğu için popüler Flutter state management paketleri kolayca port edilebilir. Jaspr'ın resmi Riverpod portu `jaspr_riverpod`'dir:

```bash
dart pub add jaspr_riverpod
```

```dart
import 'package:jaspr_riverpod/jaspr_riverpod.dart';

// Basit bir provider
final sayacProvider = StateProvider<int>((ref) => 0);

class SayacSayfasi extends StatelessComponent {
  const SayacSayfasi({super.key});

  @override
  Component build(BuildContext context) {
    final sayi = context.watch(sayacProvider);
    return div([
      p([.text('Sayaç: $sayi')]),
      button(
        onClick: () => context.read(sayacProvider.notifier).state++,
        [.text('Artır')],
      ),
    ]);
  }
}
```

## 6.4 Reactive Yapı ve UI'a Yansıma

Jaspr'da reaktivite iki düzeyde işler:

- **Component içi:** `setState()` çağrısı, o component'in `build()` metodunu yeniden çalıştırır ve DOM farkını uygular.
- **Ağaç genelinde:** `InheritedComponent` veya Riverpod gibi bir provider'daki değişiklik, ona bağımlı (`dependOn...` / `watch`) tüm component'leri yeniden build eder.

## 6.5 Sunucu → İstemci State Senkronizasyonu

Static/server modunda sayfa sunucuda render edilir; ardından istemcide "hydrate" edilir. Sunucuda yüklenen verinin istemcide de mevcut olması gerekir — Jaspr bunu framework'e gömülü olarak çözer:

### @sync anotasyonu

`State` sınıfındaki bir alanı `@sync` ile işaretlersen, değeri sunucuda pre-render sırasında serialize edilir ve istemcide `initState()` sırasında geri yüklenir:

**lib/sayac.dart**

```dart
import 'sayac.sync.dart'; // jaspr serve/build ile OTOMATİK üretilir

class Sayac extends StatefulComponent { /* ... */ }

class SayacState extends State<Sayac> with SayacStateSyncMixin {
  @sync
  int sayi = 0; // sunucudan istemciye otomatik senkronize edilir

  @override
  void initState() {
    // İstemcide: senkronize değer super.initState() sırasında atanır.
    super.initState();
    // Sunucuda: pre-render sırasında değeri burada ata.
    if (!kIsWeb) {
      sayi = sunucudanDegerGetir();
    }
  }

  void artir() => setState(() => sayi++); // istemcide normal kullanım
}
```

> [!WARNING]
> **Dikkat**
> `@sync` alanlar ilgili `.sync.dart` dosyasını import etmeyi ve üretilen `...SyncMixin`'i `with` ile eklemeyi gerektirir; dosya `jaspr serve`/`build` çalıştığında üretilir. Senkronizasyon **tek yönlüdür** (sunucu → istemci) ve yalnızca ilk render'da yapılır. İstemciden sunucuya veri göndermek için normal bir API kullan (Bölüm 8).

### @client parametreleri

`@client` component'lerinin constructor parametreleri de sunucudan istemciye otomatik serialize edilir — veri aktarmanın en pratik yoludur:

```dart
@client
class Uygulama extends StatelessComponent {
  // Hydration sırasında parametreler sunucudakiyle aynı değerleri alır.
  const Uygulama({required this.baslik, super.key});

  final String baslik;
  /* ... */
}
```

Parametre tipleri `bool`, `int`, `double`, `String` ve bunların `List`/`Map`'leri olabilir. Özel tipler için `@encoder`/`@decoder` anotasyonlarıyla `toJson`/`fromJson` tanımlarsın:

```dart
class Model {
  @decoder
  static Model fromJson(Map<String, dynamic> json) => /* ... */ Model();

  @encoder
  Map<String, dynamic> toJson() => /* ... */ {};
}
```

> [!NOTE]
> **Birden çok @client component'i arasında state paylaşımı**
> `main.client.dart`'taki `ClientApp()`'i bir `InheritedComponent` veya Riverpod `ProviderScope` ile sar: `runApp(ProviderScope(child: ClientApp()))`.

> [!TIP]
> **Alıştırma 6**
> 1. Açılır menü component'ini genişlet: seçili öğe state'te tutulsun ve menü dışı tıklamada kapanmasa da en azından seçim buton üzerinde görünsün.
> 2. İki farklı `@client` component'i oluştur ve `ProviderScope` ile ikisi arasında paylaşılan bir sayaç state'i kur.
> 3. Static modda bir projede `@sync` ile sunucuda üretilen bir "oluşturulma zamanı" değerini istemciye taşı; konsoldan iki tarafta da aynı değeri doğrula.

---

[⬅ 5. Routing](05-routing.md) | [Sonraki: 7. Formlar ➡](07-formlar.md)
