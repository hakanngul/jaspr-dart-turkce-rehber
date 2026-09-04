# 4. Dart ile Jaspr Geliştirme

Jaspr'da yazdığın her şey saf Dart'tır; ayrı bir şablon dili (JSX, template syntax vb.) yoktur. Bu bölüm, Dart dil özelliklerinin Jaspr bağlamında nasıl kullanıldığını gösterir.

## 4.1 Dart Sözdiziminin Jaspr İçinde Kullanımı

Component ağaçları sıradan Dart ifadeleri olduğu için koleksiyon kontrol akışlarını (collection-if, spread, for) doğrudan kullanabilirsin:

```dart
Component build(BuildContext context) {
  final kullaniciGirisYapti = true;
  final bildirimler = ['Mesaj 1', 'Mesaj 2'];

  return div([
    // Koşullu render: collection-if
    if (kullaniciGirisYapti)
      p([.text('Hoş geldin!')])
    else
      a(href: '/giris', [.text('Giriş yap')]),

    // Liste render: for + spread
    ul([
      for (final b in bildirimler) li([.text(b)]),
    ]),

    // Spread ile mevcut listeyi açma
    ...ekstraComponentler(),
  ]);
}
```

Bu, React'taki `{kosul && <X/>}` veya `map()` kalıplarının Dart'taki karşılığıdır — ama ayrı bir sözdizimi öğrenmeden, dilin kendi özellikleriyle.

## 4.2 Null Safety

Opsiyonel verilerle çalışırken null safety kuralları aynen geçerlidir:

```dart
class ProfilKarti extends StatelessComponent {
  const ProfilKarti({required this.isim, this.avatarUrl, super.key});

  final String isim;
  final String? avatarUrl; // null olabilir

  @override
  Component build(BuildContext context) {
    return div(classes: 'profil', [
      // Null-aware render: collection-if ile
      if (avatarUrl != null) img(src: avatarUrl!, alt: isim),
      h2([.text(isim)]),
      // Null-aware varsayılan
      p([.text(avatarUrl ?? 'Avatar yok')]),
    ]);
  }
}
```

## 4.3 Class, Inheritance ve Generics

Kendi component hiyerarşilerini ve generic component'lerini yazabilirsin:

```dart
// Generic liste component'i
class ListeKutusu<T> extends StatelessComponent {
  const ListeKutusu({required this.ogeler, required this.ogeBuilder, super.key});

  final List<T> ogeler;
  final Component Function(T oge) ogeBuilder;

  @override
  Component build(BuildContext context) {
    return ul([
      for (final oge in ogeler) li([ogeBuilder(oge)]),
    ]);
  }
}

// Kullanım:
ListeKutusu<String>(
  ogeler: ['Elma', 'Armut'],
  ogeBuilder: (meyve) => .text(meyve),
)
```

## 4.4 Async/Await, Future ve Stream

İstemci tarafında asenkron işlemler `StatefulComponent` içinde yönetilir:

```dart
class VeriSayfasi extends StatefulComponent {
  const VeriSayfasi({super.key});
  @override
  State<VeriSayfasi> createState() => _VeriSayfasiState();
}

class _VeriSayfasiState extends State<VeriSayfasi> {
  String? sonuc;
  bool yukleniyor = false;

  Future<void> veriGetir() async {
    setState(() => yukleniyor = true);
    try {
      await Future.delayed(Duration(seconds: 1)); // API çağrısı simülasyonu
      sonuc = 'Veri geldi!';
    } finally {
      setState(() => yukleniyor = false);
    }
  }

  @override
  Component build(BuildContext context) {
    return div([
      button(onClick: veriGetir, [.text('Veri Getir')]),
      if (yukleniyor) p([.text('Yükleniyor...')]),
      if (sonuc != null) p([.text(sonuc!)]),
    ]);
  }
}
```

> [!NOTE]
> **Sunucuda async build**
> Static/server modunda `AsyncBuilder` ve `AsyncStatelessComponent` ile build metodunun kendisinde `await` kullanabilirsin (yalnızca sunucuda çalışır). Detaylar Bölüm 8'de.

## 4.5 Dependency Yönetimi

Jaspr projeleri normal Dart projeleridir; bağımlılıklar `pubspec.yaml` üzerinden yönetilir:

```bash
dart pub add jaspr_router     # routing
dart pub add http             # HTTP istekleri
dart pub add jaspr_riverpod   # state management
dart pub add jaspr_test --dev # test
```

> [!WARNING]
> **Dikkat: Platform uyumluluğu**
> İstemcide çalışacak koda `dart:io` kullanan bir paket ekleyemezsin (ör. dosya sistemi paketleri). İstemci kodunda HTTP için `package:http` (fetch tabanlı) gibi web uyumlu paketler seç. pub.dev'de paketin "platforms" rozeti bunu gösterir. Jaspr paketlerini `#jaspr` etiketiyle filtreleyebilirsin.

> [!TIP]
> **Alıştırma 4**
> 1. Bir `Kitap` model sınıfı yaz (isim, yazar, opsiyonel kapak URL'si) ve generic `ListeKutusu` ile bir kitap listesi render et.
> 2. Sahte gecikmeli bir Future ile "yükleniyor → veri geldi" akışını bir StatefulComponent'te uygula.
> 3. `dart pub add` ile `http` paketini ekle ve `pubspec.yaml`'daki değişikliği incele.

---

[⬅ 3. Temeller](03-temeller.md) | [Sonraki: 5. Routing ➡](05-routing.md)
