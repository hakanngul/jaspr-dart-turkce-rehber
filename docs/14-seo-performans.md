# 14. SEO ve Performans

## 14.1 Pre-Rendering = SEO Temeli

Static ve server modlarında tarayıcı tam HTML'i hemen alır; arama motorları içeriği görmek için JS çalıştırmak zorunda kalmaz. Bu, Jaspr'ın Flutter Web'e karşı en büyük avantajıdır.

## 14.2 Meta Tag Yönetimi

Site geneli meta bilgileri `Document` üzerinde (server/static):

```dart
runApp(Document(
  title: 'Benim Sitem',
  lang: 'tr',
  meta: {
    'description': 'Sitenin arama sonuçlarında görünen kısa açıklaması',
    'keywords': 'dart, jaspr, web',
    'og:title': 'Sosyal önizleme başlığı',
    'og:image': 'https://ornek.com/kapak.png',
  },
  body: App(),
));
```

Sayfa özelinde geçersiz kılmak için ağacın **herhangi bir yerinde** `Document.head()` kullan:

```dart
Document.head(
  title: 'Makale Başlığı',
  meta: {'description': 'Makalenin özeti...'},
)
```

Override kuralları: `<title>` ve `<base>` birbirinin yerine geçer; `<meta>` etiketleri aynı `name`'e göre eşleşir; daha derindeki/sonraki component kazanır.

## 14.3 Sitemap (Static Mod)

```bash
jaspr build --sitemap-domain https://ornek.com
```

Build çıktısına `sitemap.xml` eklenir. Route bazında özelleştirme:

```dart
Route(
  path: '/ozet',
  settings: RouteSettings(changeFreq: ChangeFreq.weekly, priority: 0.9),
  builder: (context, state) => OzetSayfasi(),
)
```

## 14.4 Performans Optimizasyonları

1. **Hedefli hydration:** `@client`'i ağaçta mümkün olan en aşağıya taşı; yalnızca etkileşimli parçalar JS yüklesin.
2. **Otomatik code splitting:** Server/static modda her `@client` component'i otomatik olarak ayrı JS parçasına bölünür — çoğu durumda ek iş gerekmez.
3. **Lazy routes:** Client modunda/ince ayar için `Route.lazy` + deferred import (Bölüm 5.7).
4. **Deferred import ile manuel yükleme:** Ağır bir kütüphaneyi (ör. grafik, editör) gerektiğinde yükle:
   ```dart
   import 'components/agir_editor.dart' deferred as editor;

   // initState'te:
   editor.loadLibrary().then((_) => setState(() => yuklendi = true));
   ```
5. **Resim optimizasyonu:** `alt` her yerde; `.webp` tercih et; kritik görselleri `<link rel="preload">` ile önden yükle.
6. **Font optimizasyonu:** Yalnızca kullandığın ağırlıkları yükle, `display=swap` kullan.

> [!TIP]
> **Alıştırma 14**
> 1. Projendeki her sayfaya özgün `title` ve `description` ekle (`Document.head` ile).
> 2. Static modda `--sitemap-domain` ile build alıp `sitemap.xml`'i incele.
> 3. Tarayıcı DevTools Network sekmesinde, `@client`'i aşağı taşımanın JS boyutuna etkisini ölç.

---

[⬅ 13. JavaScript Entegrasyonu](13-javascript-entegrasyon.md) | [Sonraki: 15. Authentication ➡](15-auth.md)
