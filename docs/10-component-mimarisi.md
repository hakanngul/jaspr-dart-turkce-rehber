# 10. Component Mimarisi ve Proje Organizasyonu

## 10.1 Yeniden Kullanılabilir Component Tasarımı

İyi bir Jaspr component'i şu özellikleri taşır:

- **Tek sorumluluk:** Bir component tek bir UI parçasından sorumlu olsun (Kart, Buton, Avatar...).
- **Parametrelerle yapılandırma:** Değişken her şeyi constructor parametresi yap; içeride sabit bırakma.
- **Slot deseni:** İçerik alanları için `Component` veya `List<Component>` parametreleri kullan (Flutter'daki `child`/`children` gibi).
- **Const constructor:** Mümkünse `const` yap — performans ve tutarlılık sağlar.

```dart
class UyariKutusu extends StatelessComponent {
  const UyariKutusu({required this.baslik, required this.cocuklar, super.key});

  final String baslik;
  final List<Component> cocuklar; // slot deseni

  @override
  Component build(BuildContext context) {
    return div(classes: 'uyari-kutusu', [
      strong([.text(baslik)]),
      ...cocuklar,
    ]);
  }
}

// Kullanım:
UyariKutusu(
  baslik: 'Dikkat',
  cocuklar: [
    p([.text('Bu işlem geri alınamaz.')]),
    button(onClick: onayla, [.text('Onayla')]),
  ],
)
```

## 10.2 Component Lifecycle

StatefulComponent yaşam döngüsü Flutter ile aynıdır:

| Metod | Ne zaman çağrılır | Tipik kullanım |
|-------|-------------------|----------------|
| `preloadState()` | Yalnızca sunucuda, `initState` öncesi (PreloadStateMixin ile) | Async veri yükleme |
| `initState()` | Component ilk oluştuğunda, bir kez | Başlangıç state'i, abonelikler |
| `didChangeDependencies()` | Bağımlı olunan InheritedComponent değiştiğinde | Provider'a tepki verme |
| `didUpdateComponent()` | Üst component yeniden build edip yeni parametreler gönderdiğinde | Parametre değişimine tepki |
| `build()` | Her render'da | UI tanımı |
| `dispose()` | Component ağaçtan kaldırılırken | StreamSubscription iptali, temizlik |

> [!WARNING]
> **Yaygın Hata**
> `initState` içinde bir `StreamSubscription` başlatıp `dispose`'da iptal etmemek bellek sızıntısı yapar. Kural: `initState`'te açtığın her kaynağı `dispose`'da kapat.

## 10.3 Büyük Projelerde Organizasyon

Önerilen katmanlı yapı (Clean Architecture'dan ilhamla):

```text
lib/
├── main.server.dart / main.client.dart
├── app.dart                  # Router + kök yerleşim
├── core/
│   ├── theme.dart            # Renkler, stiller (@css)
│   └── utils/                # Validators, formatlayıcılar (saf Dart)
├── data/
│   ├── models/               # Post, User... (@encoder/@decoder ile)
│   └── services/             # API istemcileri, repository'ler
├── state/                    # Provider tanımları (jaspr_riverpod)
├── components/               # Paylaşılan UI parçaları (Kart, Buton...)
│   └── layout/               # Header, Footer, Shell
└── pages/                    # Route'lara karşılık gelen sayfalar
    ├── home.dart
    └── posts/
        ├── post_list.dart
        └── post_detail.dart
```

Kurallar:

- **Bağımlılık yönü tek yönlüdür:** pages → components → state → data → core. Ters yönde import yapma.
- **Servisler UI bilmez:** `data/services` içindeki sınıflar component import etmez; saf Dart kalırlar, böylece birim testi kolaydır.
- **Sunucu/istemci ayrımı:** İki ortamda da çalışan modeller `data/models` altında; yalnızca sunucuya ait kod (ör. veritabanı) ayrı dosyada tutulur ve client scope'a import edilmez.

> [!TIP]
> **Alıştırma 10**
> 1. Mevcut projendeki tek dosyalık kodu yukarıdaki klasör yapısına böl.
> 2. `UyariKutusu` desenini kullanarak "Bildirim", "Kart", "BoşDurum" component'lerinden oluşan küçük bir UI kit'i yaz.
> 3. Bir `StreamSubscription` başlatıp `dispose`'da doğru şekilde kapatan component yaz.

---

[⬅ 9. Rendering Modları](09-rendering-modlari.md) | [Sonraki: 11. CSS ve UI ➡](11-css-ui.md)
