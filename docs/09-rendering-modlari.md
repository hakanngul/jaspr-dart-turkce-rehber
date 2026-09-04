# 9. Rendering Modları: SSR, SSG ve Client

## 9.1 Jaspr'ın Rendering Mimarisi

Jaspr full-stack bir framework'tür: kodunun bir kısmı sunucuda, bir kısmı istemcide (tarayıcı) çalışır. Üç rendering modu vardır:

|  | static | server | client |
|--|--------|--------|--------|
| HTML/CSS üretimi | Evet | Evet | Hayır |
| Pre-rendering | Build zamanında | Her istekte | Yok |
| İstemci etkileşimi | Evet | Evet | Evet |
| Sunucu gerektirir | Hayır | Evet | Hayır |
| Statik hosting'e deploy | Evet | Hayır | Evet |
| Özel backend | Hayır | Evet | Hayır |

## 9.2 SSR Nedir? (Server-Side Rendering)

**server** modunda Jaspr, gelen her istek için component'lerini sunucuda çalıştırıp tam HTML üretir. Tarayıcı içeriği anında görür; ardından JS yüklenip hydration ile sayfa etkileşimli hale gelir. Giriş yapmış kullanıcıya özel içerik, sık değişen veriler gibi durumlar için doğru seçimdir. Deploy için Dart çalıştırabilen bir ortam (veya Docker) gerekir.

## 9.3 Static Site Generation (SSG)

**static** modunda `jaspr build`, sitenin tüm route'larını build zamanında render edip statik `.html` dosyaları üretir. Örneğin `/`, `/about`, `/contact` route'ları `index.html`, `about/index.html`, `contact/index.html` olarak çıkar. Sunucu gerekmez; herhangi bir statik hosting'e koyabilirsin.

`jaspr_router` kullanıyorsan tanımlı tüm route'lar otomatik üretilir. Veriden türeyen dinamik sayfalar için route'ları döngüyle tanımla:

```dart
Router(
  routes: [
    for (var post in posts) // posts build'den önce yüklenir
      Route(path: '/posts/${post.id}', builder: (context, state) => PostPage(post)),
  ],
)
```

> [!WARNING]
> **Dikkat**
> Static modda `:param` şablonlu route'lar build sırasında çözülemediği için desteklenmez; her sayfa için ayrı route üretmelisin. Elle kurulumda `ServerApp.requestRouteGeneration('/yol')` çağrısıyla route bildirebilirsin (yalnızca sunucu kodunda).

## 9.4 Client-Side Hydration

**Hydration**, sunucuda üretilmiş statik HTML'e istemcide event handler'ları bağlayarak sayfayı etkileşimli hale getirme işlemidir. Jaspr'da yaşam döngüsü şöyledir (static/server modu):

1. Sunucu component ağacını bir kez build edip HTML üretir.
2. Tarayıcı HTML'i hemen gösterir (henüz etkileşimsiz).
3. Derlenmiş JS yüklenir; uygulama istemcide yeniden başlar.
4. `main.client.dart`'taki `ClientApp()`, `@client` component'lerini bulur ve mevcut DOM'a "bağlanır".

**Hedefli hydration** önemli bir optimizasyondur: tüm ağacı istemciye taşımak yerine yalnızca `@client` işaretli component'ler hydrate edilir. Bu yüzden `@client` anotasyonunu ağaçta mümkün olduğunca aşağı taşı (örn. blog yazısındaki tek bir "Beğen" butonu için tüm sayfayı istemciye taşıma).

```dart
@client
class BegeniButonu extends StatefulComponent {
  const BegeniButonu({super.key});
  @override
  State<BegeniButonu> createState() => _BegeniButonuState();
}
// Yalnızca bu buton istemcide etkileşimli olur;
// sayfanın geri kalanı saf HTML olarak kalır.
```

## 9.5 Sunucu ve İstemci Scope'ları

Static/server modunda uygulaman iki örtüşen parçadan oluşur:

- **Server scope:** `main.server.dart`'tan başlayan, sunucuda render edilen her şey.
- **Client scope:** `@client` component'lerinden aşağı doğru, hem sunucuda hem istemcide render edilen alt ağaçlar.

Bir component'in hangi scope'ta olduğu, hangi kütüphaneleri kullanabileceğini belirler:

| Ortam | Kullanılabilir | Kullanılamaz |
|-------|----------------|--------------|
| Sunucu | `dart:io`, dosya/veritabanı paketleri, `package:jaspr/server.dart` | `dart:js_interop`, `package:web` |
| İstemci | `package:web`, `dart:js_interop`, `package:jaspr/client.dart` | `dart:io` vb. |
| İkisi de | `package:universal_web`, saf Dart paketleri | — |

> [!WARNING]
> **Yaygın Hata: yanlış import**
> Sunucu scope'undaki component'e `dart:js_interop` veya istemci scope'undakine `dart:io` import etmek derleme hatası verir. Çözümler: (1) `package:universal_web` kullan — iki ortamda da çalışır (sunucuda mock'lar); (2) `@Import` anotasyonuyla kütüphaneyi tek ortama sınırla; (3) Dart'ın koşullu import mekanizmasını kullan. `jaspr_lints` paketindeki `unsafe_imports` kuralı bu hataları önceden yakalar; VS Code eklentisi de her component'in üzerinde "Server Scope | Client Scope" ipucu gösterir.

## 9.6 SEO Açısından Etkileri ve Mod Seçimi

- Pre-render edilmiş HTML (static/server) arama motorlarına içeriği doğrudan sunar — SEO'nun temelidir. Client modunda içerik JS çalışana kadar HTML'de yoktur.
- **Blog/dokümantasyon/tanıtım:** static. İçerik odaklı sitelerde ayrıca `jaspr_content` paketine bak (markdown tabanlı içerik, Bölüm 18.1).
- **Girişli, kişiselleştirilmiş uygulama:** server.
- **Login arkası panel/dashboard:** client (SEO gereksiz, sunucu maliyeti sıfır).
- Static modda SPA bile kurabilirsin: `index.html` ve `styles.css` yazmadan her şeyi Dart'ta yazıp build'de HTML/CSS'e çevirebilirsin.

> [!TIP]
> **Alıştırma 9**
> 1. Aynı component ağacını önce client, sonra static modda çalıştır; "Sayfa Kaynağını Görüntüle" ile farkı karşılaştır.
> 2. Static modda beş yazılık sahte bir blog için döngüyle route üret ve `jaspr build` çıktısındaki HTML dosyalarını incele.
> 3. Bir sayfada yalnızca tek bir butonu `@client` yap; build çıktısındaki JS boyutunu tüm sayfayı @client yaptığın durumla karşılaştır.

---

[⬅ 8. API ve Backend](08-api-backend.md) | [Sonraki: 10. Component Mimarisi ➡](10-component-mimarisi.md)
