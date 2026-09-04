# Ek C. Best Practices ve Proje Checklist'i

## Best Practices

1. **@client'i aşağı taşı:** Yalnızca gerçekten etkileşimli parçaları istemciye gönder.
2. **Const kullan:** Component'lerde ve `Styles`'ta mümkünse `const` — boyut ve performans kazandırır.
3. **Doğru mod seçimi:** İçerik sitesi → static; dinamik/kişisel → server; panel → client.
4. **jaspr_lints kur:** Özellikle `unsafe_imports` kuralı scope hatalarını erken yakalar.
5. **VS Code eklentisini kullan:** Scope ipuçları (Server/Client) kafa karışıklığını önler.
6. **Saf Dart katmanı:** İş mantığını component'lerden ayır; test edilebilirliği koru.
7. **Gizli anahtarları sunucuda tut:** İstemciye derlenen her şey herkese açıktır.
8. **SEO üçlüsü:** Her sayfada özgün title + description + og:image.
9. **Sürüm takibi:** 0.x sürümlerinde yükseltmeden önce release notlarını oku; `jaspr migrate`'i kullan.

## Gerçek Proje Geliştirme Checklist'i

| Aşama | Kontroller |
|-------|------------|
| Başlangıç | Mod seçildi (static/server/client), şablon oluşturuldu, jaspr_lints eklendi |
| Mimari | Klasör yapısı kuruldu (core/data/state/components/pages), bağımlılık yönü tek yönlü |
| UI | Responsive media query'ler, erişilebilirlik (alt, label, kontrast) |
| Veri | Loading/error/data durumları her veri ekranında; @sync/PreloadStateMixin doğru kurulum |
| SEO | Sayfa başına title/description, sitemap.xml, og etiketleri, robots.txt |
| Güvenlik | Auth sunucuda doğrulanıyor, cookie'ler HttpOnly+Secure, gizli anahtar istemcide yok |
| Test | Kritik component'ler testComponents ile; SSR yanıtları testServer ile |
| Performans | @client aşağıda, ağır kütüphaneler deferred, resimler webp/optimize |
| Deploy | jaspr build temiz geçiyor, CI workflow'u kuruldu, rollback planı var |

---

[⬅ Ek B — Sık Yapılan Hatalar](ek-b-sik-hatalar.md) | [Sonraki: Ek D — Klasör Yapısı ve Roadmap ➡](ek-d-klasor-roadmap.md)
