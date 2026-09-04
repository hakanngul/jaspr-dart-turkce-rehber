# 1. Jaspr Nedir?

## 1.1 Tanım ve Amaç

**Jaspr**, tamamen Dart ile yazılmış modern, full-stack bir web framework'üdür. Flutter'a görünüş ve his olarak çok benzeyen bir component modeli sunar; ancak Flutter Web'in aksine canvas'a piksel çizmek yerine **gerçek HTML, DOM ve CSS** üretir. Yani yazdığın Dart kodu, tarayıcının doğal olarak anladığı standart web teknolojilerine dönüşür.

Jaspr'ın resmi sitesindeki üç temel soruya verdiği cevap şöyledir:

- **Neden?** Flutter gibi görünen ve hissettiren, fakat normal HTML ve CSS render eden bir web framework'ü oluşturmak için.
- **Kimin için?** Dart ile her tür web sitesi (özellikle Flutter Web'e uygun olmayanlar) geliştirmek isteyen, ağırlıklı olarak Flutter geliştiricileri için.
- **Ne?** Dart'ın web ve sunucudaki sınırlarını zorlayan, baştan sona düşünülmüş bir full-stack web framework'ü.

Jaspr'ın öne çıkan özellikleri:

- **Tanıdık:** Flutter widget'larına benzer component modeli (`StatelessComponent`, `StatefulComponent`, `InheritedComponent`).
- **Güçlü:** Server-side rendering (SSR) kutudan çıkar çıkmaz hazır.
- **Kolay:** Sunucu ve istemci arasında component state'ini otomatik senkronize eder (`@sync`).
- **Hızlı:** Sadece gereken yerlerde optimize DOM güncellemeleri yapar.
- **Esnek:** Sunucuda, istemcide veya her ikisinde birden çalışabilir; kurulumu manuel ya da otomatik yapabilirsin.

## 1.2 Dart ve Flutter ile İlişkisi

Jaspr, Flutter'ın **alternatifi** değil, **tamamlayıcısıdır**. İlişkiyi şöyle özetleyebiliriz:

- **Dart** dildir; hem Flutter hem Jaspr bu dille yazılır.
- **Flutter**, mobil/masaüstü/web için canvas tabanlı bir UI toolkit'idir. Web'de Skia/CanvasKit ile piksel çizer.
- **Jaspr**, Flutter'ın component modelini ödünç alır ama render katmanı tamamen web'e özgüdür: HTML etiketleri, DOM ağacı ve CSS.

İlginç bir kanıt: `dart.dev` ve `docs.flutter.dev` sitelerinin ikisi de Jaspr ile geliştirilmiştir. Bu, Jaspr'ın metin ağırlıklı, içerik odaklı sitelerde ne kadar yetkin olduğunun iyi bir göstergesidir.

## 1.3 Hangi Problemleri Çözer?

Flutter ekibi, kendi dokümantasyonunda şunu açıkça belirtir: *"Flutter Web, web siteleri değil web uygulamaları içindir."* Metin ağırlıklı, akış tabanlı, statik içerikler (blog yazıları gibi) web'in doküman merkezli modelinden faydalanır; Flutter'ın uygulama merkezli modeli bu işe uygun değildir. Jaspr tam olarak bu boşluğu doldurur:

- **SEO problemi:** Flutter Web, içeriği canvas içine çizdiği için arama motorları içeriği göremez. Jaspr, sunucuda gerçek HTML ürettiği için SEO dostu siteler sağlar.
- **İlk yükleme hızı:** Flutter Web'de büyük bir JavaScript bundle'ı yüklenmeden ekran boş kalır. Jaspr'da pre-render edilmiş HTML anında görüntülenir (First Contentful Paint çok hızlıdır).
- **Web standartlarından kopukluk:** Flutter Web'de metin seçimi, tarayıcı eklentileri, erişilebilirlik araçları ve mevcut CSS ekosistemiyle uyum sorunları yaşanır. Jaspr gerçek DOM kullandığı için bunların hepsi doğal olarak çalışır.
- **Full-stack tutarlılık:** Frontend ve backend'i aynı dilde (Dart) yazarsın; modelleri, validasyon kodunu ve iş mantığını iki taraf arasında paylaşabilirsin.

## 1.4 Hangi Projelerde Tercih Edilmeli?

| Proje Tipi | Uygunluk | Önerilen Mod |
|------------|----------|--------------|
| Blog, dokümantasyon, tanıtım sitesi, portföy | Çok uygun | static |
| Kurumsal site, landing page, SEO kritik sayfalar | Çok uygun | static / server |
| Kullanıcı girişi olan dinamik web uygulaması | Uygun | server |
| Dashboard, admin paneli, SPA | Uygun | client |
| Ağır grafik/animasyon, oyun benzeri UI | Uygun değil | Flutter Web düşün |
| Mobil uygulama | Uygun değil | Flutter kullan |

## 1.5 Avantajlar ve Dezavantajlar

### Avantajlar

- Flutter bilen biri için öğrenme eğrisi çok sığdır; component modeli neredeyse birebir aynıdır.
- Gerçek HTML/CSS üretir: SEO, erişilebilirlik, tarayıcı uyumluluğu doğal olarak gelir.
- Üç rendering modu (static / server / client) ile Next.js benzeri esneklik sunar.
- Sunucu-istemci state senkronizasyonu (`@sync`, `@client` parametreleri) framework'e gömülüdür.
- Tek dil (Dart) ile frontend + backend; modeller ve validasyonlar paylaşılır.
- Mevcut CSS ekosistemi (Tailwind, Bulma, Sass) ve Dart paketleriyle (shelf, Serverpod, dart_frog) entegre olur.

### Dezavantajlar

- Ekosistem gençtir; React/Next.js'e kıyasla hazır paket ve topluluk kaynağı azdır.
- Hazır stillendirilmiş component kütüphanesi (Material/Cupertino karşılığı) **yoktur**; UI'ı CSS ile kendin kurarsın.
- Türkçe kaynak ve video içerik neredeyse yoktur (bu rehberin varlık sebebi de budur).
- Sunucu + istemci "scope" ayrımını anlamak başlangıçta kafa karıştırabilir (Bölüm 9'da detaylı anlatılır).
- 0.x sürümündedir; major güncellemelerde kırıcı değişiklikler olabilir (örn. 0.22'deki entrypoint değişikliği).

## 1.6 Jaspr vs Flutter Web

| Özellik | Jaspr | Flutter Web |
|---------|-------|-------------|
| Render hedefi | Gerçek HTML/DOM + CSS | Canvas (piksel çizimi) |
| SEO | Mükemmel (pre-render edilmiş HTML) | Zayıf |
| Layout | HTML elementleri + CSS (flexbox, grid) | Widget'lar (Row, Column, Stack) |
| Stil | CSS / Styles sınıfı / CSS framework'leri | Widget parametreleri, tema |
| Hazır component | Yok (HTML etiketleri var) | Zengin Material/Cupertino seti |
| İlk yükleme | Hızlı (küçük JS, pre-render) | Yavaş (büyük bundle) |
| Routing | Çok sayfalı (server) veya tek sayfalı (client) | Sadece tek sayfalı |
| İdeal kullanım | Web siteleri, içerik, SEO'lu uygulamalar | Uygulama benzeri web arayüzleri |

> [!NOTE]
> **Tasarım Tercihi: Neden Row/Column yok?**
> Jaspr, Flutter'ın `Row`, `Column`, `Stack` gibi layout primitive'lerini bilerek kopyalamaz. Bunun yerine `div` gibi standart HTML elementlerini CSS ile kullanırsın. Amaç, web platformunun kendi güçlü yanlarını gizlememek ve geliştiriciyi web'e özgü teknolojilere (CSS) yakın tutmaktır. Aynı sebepten `Component.text()` stil parametresi almaz; stillendirme her zaman CSS ile yapılır.

## 1.7 Klasik Frontend Framework'leriyle Karşılaştırma

- **React/Next.js ile benzerlik:** Component tabanlı yapı, SSR/SSG desteği, hydration kavramı. Jaspr'ın `@client` component'leri Next.js'in "islands" yaklaşımına benzer.
- **Temel fark:** JSX yerine Dart component ağacı yazarsın; tip güvenliği Dart'ın güçlü tip sistemi sayesinde çok daha katıdır. CSS-in-Dart `Styles` API'si ile stiller bile tip kontrolünden geçer.
- **Klasik HTML/JS ile fark:** Elle DOM manipülasyonu yerine bildirimsel (declarative) component modeli kullanırsın; state değiştiğinde Jaspr DOM'u senin yerine günceller.

> [!NOTE]
> **Bilgi**
> Jaspr, Flutter Web uygulamalarını **gömme** (element embedding) ve web destekli Flutter plugin'lerini (`shared_preferences`, `firebase` gibi) kullanma desteğine de sahiptir. `pubspec.yaml`'a `jaspr: { flutter: plugins }` ekleyerek etkinleştirilir.

> [!TIP]
> **Alıştırma 1**
> 1. JasprPad'i tarayıcıda aç ve hazır "Weather Api" örneğini çalıştır. Kodda `StatefulComponent` ve `State` sınıflarını bul.
> 2. Aynı örnekteki bir metni değiştirip tekrar çalıştır; çıktının gerçek HTML olduğunu tarayıcının "kaynağı görüntüle" özelliğiyle doğrula.
> 3. Kendi projende kullanmayı düşündüğün bir web fikrini yaz ve Tablo 1.1'e göre hangi rendering modunun uygun olacağını gerekçesiyle not et.

---

[⬅ 0. Yol Haritası](00-yol-haritasi.md) | [Sonraki: 2. Kurulum ve Başlangıç ➡](02-kurulum.md)
