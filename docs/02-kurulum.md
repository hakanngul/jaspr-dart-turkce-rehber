# 2. Kurulum ve Başlangıç

## 2.1 Gereksinimler

- **Dart SDK 3.10+** (dot-shorthand sözdizimi ve güncel Jaspr sürümü için önerilir; minimum destek için pub.dev'deki jaspr sayfasındaki SDK kısıtına bak).
- Bir metin editörü — **VS Code + Jaspr Extension** önerilir. Eklenti, her component'in sunucuda mı istemcide mi render edildiğini editörde gösterir; bu, Jaspr öğrenirken en büyük kolaylıktır.
- Terminal erişimi (macOS/Linux/Windows).
- Flutter SDK zorunlu değildir; yalnızca Flutter embedding veya Flutter plugin'leri kullanacaksan gerekir.

## 2.2 Dart Kurulumu

Dart kurulu değilse `dart.dev/get-dart` adresindeki talimatları izle. Kurulumu doğrula:

```bash
dart --version
# Dart SDK version: 3.10.0 (veya üzeri) görmelisin
```

## 2.3 Jaspr CLI Kurulumu

Jaspr komut satırı aracı (CLI), proje oluşturma, geliştirme sunucusu ve build işlemlerini yönetir. **Güncel (0.23+)** önerilen kurulum yöntemi AOT derlenmiş hızlı CLI'dir:

```bash
# Güncel kullanım (önerilen, daha hızlı AOT executable):
dart install jaspr_cli

# Eski kullanım (hâlâ çalışır):
dart pub global activate jaspr_cli

# Not: dart install kullanacaksan önce eski global aktivasyonu kaldır:
dart pub global deactivate jaspr_cli
```

Doğrulama:

```bash
jaspr --version
jaspr --help
```

> [!WARNING]
> **Yaygın Hata: "jaspr: command not found"**
> Pub cache bin dizini PATH'te değildir. macOS/Linux: `export PATH="$PATH:$HOME/.pub-cache/bin"` satırını `~/.zshrc` / `~/.bashrc` dosyana ekle. Windows: `%LOCALAPPDATA%\Pub\Cache\bin` dizinini sistem PATH'ine ekle.

## 2.4 Yeni Proje Oluşturma

```bash
jaspr create my_website
cd my_website
```

CLI sana bir şablon seçtirir. Şablonlar rendering modlarına karşılık gelir:

| Şablon | Mod | Ne zaman? |
|--------|-----|-----------|
| Static Site | static | Genel amaçlı (önerilen başlangıç). Blog, tanıtım, dokümantasyon. |
| Server Rendered Site | server | Backend entegrasyonu, auth, sık değişen dinamik içerik. |
| Single Page Application | client | Dashboard, admin paneli gibi uygulama benzeri siteler. |
| Embedded Flutter Site | static + Flutter | Flutter widget demosu gömülü siteler. |
| Custom Backend Site | server + shelf | Kendi API'ni yazacağın full-stack projeler. |

> [!NOTE]
> **Bilgi**
> Şablon seçimi seni kalıcı olarak kilitlemez; `pubspec.yaml`'daki `jaspr.mode` ayarını değiştirerek modu sonra da değiştirebilirsin. VS Code kullanıyorsan aynı işi komut paletindeki **Jaspr: New Project** komutuyla da yapabilirsin.

## 2.5 Proje Klasör Yapısı (0.22+)

```text
my_website/
├── lib/
│   ├── main.server.dart          # Sunucu giriş noktası (static/server modu)
│   ├── main.server.options.dart  # OTOMATİK ÜRETİLİR — elle düzenleme!
│   ├── main.client.dart          # İstemci giriş noktası (tüm modlar)
│   ├── main.client.options.dart  # OTOMATİK ÜRETİLİR — elle düzenleme!
│   ├── app.dart                  # Kök component
│   ├── components/               # Yeniden kullanılabilir component'ler
│   └── pages/                    # Sayfa component'leri
├── web/                          # Statik dosyalar (resim, css, favicon...)
├── pubspec.yaml                  # Bağımlılıklar + jaspr konfigürasyonu
└── build/jaspr/                  # Build çıktısı (jaspr build sonrası)
```

`lib/main.server.dart` (sunucu giriş noktası):

```dart
// Sunucuya özgü Jaspr import'u
import 'package:jaspr/server.dart';

// Bu dosya Jaspr tarafından otomatik üretilir, silme veya düzenleme.
import 'main.server.options.dart';

void main() {
  // Sunucu ortamını üretilen varsayılan seçeneklerle başlatır.
  Jaspr.initializeAll(options: defaultServerOptions);

  // Uygulamayı sunmaya başlar.
  runApp(
    Document(
      title: 'Benim Jaspr Sitem',
      body: App(),
    ),
  );
}
```

`lib/main.client.dart` (istemci giriş noktası):

```dart
// İstemciye özgü Jaspr import'u (eski adı: package:jaspr/browser.dart)
import 'package:jaspr/client.dart';

import 'main.client.options.dart';

void main() {
  Jaspr.initializeApp(options: defaultClientOptions);

  // @client ile işaretlenmiş tüm component'leri otomatik bulur
  // ve hydrate eder (etkileşimli hale getirir).
  runApp(const ClientApp());
}
```

> [!WARNING]
> **Eski Kullanım / Güncel Kullanım (0.22 değişikliği)**
> 0.22 öncesinde tek bir `lib/main.dart` ve `package:jaspr/browser.dart` vardı. 0.22 ile sunucu ve istemci giriş noktaları ayrıldı (`main.server.dart` / `main.client.dart`), `browser.dart` `client.dart` olarak yeniden adlandırıldı. Eski projende `jaspr migrate` komutunu çalıştırarak otomatik geçiş yapabilirsin.

## 2.6 İlk Uygulamayı Çalıştırma

```bash
jaspr serve
```

Bu komut bir geliştirme sunucusu başlatır (varsayılan `http://localhost:8080`). Kodda değişiklik yaptığında tarayıcı otomatik yenilenir (hot reload). Durdurmak için `Ctrl+C`.

VS Code kullanıyorsan `F5` ile debug başlatabilirsin. Static/server modunda **iki** ayrı debug oturumu açılır: biri sunucu, biri istemci için. İkisi arasında debug kenar çubuğundan geçiş yaparsın.

## 2.7 Development ve Production Mantığı

- **Geliştirme (`jaspr serve`):** Static modda bile sayfalar istek anında (on-demand) sunucuda render edilir. `kDebugMode` sabiti `true`'dur; geliştirmeye özel kod yazabilirsin.
- **Production (`jaspr build`):** Çıktı `build/jaspr` klasörüne yazılır. Static modda tüm sayfalar `.html` dosyalarına dönüştürülür; server modunda `app` (Linux) çalıştırılabilir dosyası + `web/` asset klasörü üretilir; client modunda derlenmiş JS + index.html üretilir.

```bash
# Production build al
jaspr build

# Çıktıyı incele
ls build/jaspr
```

> [!TIP]
> **Alıştırma 2**
> 1. CLI ile `static` şablonunda bir proje oluştur, `jaspr serve` ile çalıştır ve tarayıcıda aç.
> 2. `app.dart` içindeki karşılama metnini değiştir; hot reload'un çalıştığını gözlemle.
> 3. `jaspr build` çalıştır ve `build/jaspr` içindeki üretilen `index.html` dosyasını açıp kaynağını incele — Dart kodunun gerçek HTML'e dönüştüğünü gör.

---

[⬅ 1. Jaspr Nedir?](01-jaspr-nedir.md) | [Sonraki: 3. Jaspr Temelleri ➡](03-temeller.md)
