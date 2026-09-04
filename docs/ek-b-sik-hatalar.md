# Ek B. Sık Yapılan Hatalar ve Çözümleri

| Hata | Neden | Çözüm |
|------|-------|-------|
| `jaspr: command not found` | pub cache bin PATH'te değil | `~/.pub-cache/bin` dizinini PATH'e ekle |
| Sunucu component'inde `dart:js_interop` derleme hatası | Scope karışıklığı | `package:universal_web` kullan veya `@Import`/koşullu import |
| Form gönderiminde sayfa yenileniyor | `preventDefault()` unutulmuş | `events: {'submit': (e) => e.preventDefault()}` |
| `Router.of(context)` null dönüyor | Router istemci ağacında yok (multi-page kurulum) | Navigasyon için `Link` kullan |
| Static build'de `:param` route'u üretilmiyor | SSG'de şablon route desteklenmez | Her sayfa için döngüyle ayrı route tanımla |
| `@sync` alanı istemcide boş geliyor | `.sync.dart` import'u / mixin eksik | `import 'dosya.sync.dart';` + `with ...SyncMixin` |
| `initState`'te sunucu API'si patlıyor | İstemcide de çalışıyor | `if (!kIsWeb)` ile sar |
| Pre-render HTML'de istenmeyen boşluklar | Jaspr'ın otomatik girinti biçimlendirmesi | İlgili kısmı `<span>` ile sar (span'a biçimlendirme uygulanmaz) |
| Eski dokümandaki `text('...')` deprecated uyarısı | 0.22 API değişikliği | `.text('...')` kullan veya `jaspr migrate` çalıştır |
| İstemcide `dart:io` paketi çalışmıyor | Web'de dart:io yok | Web uyumlu paket seç (pub.dev platform rozetine bak) |
| Standalone CSS build hatası | `@css` dosyasında web import'u | Stil tanımlarını web kütüphanesi import etmeyen dosyalarda tut |

---

[⬅ Ek A — Cheat Sheet](ek-a-cheat-sheet.md) | [Sonraki: Ek C — Best Practices ➡](ek-c-best-practices.md)
