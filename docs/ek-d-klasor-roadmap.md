# Ek D. Proje Klasör Yapısı ve Öğrenme Roadmap'i

## Önerilen Proje Klasör Yapısı (Tam Örnek)

```text
proje/
├── lib/
│   ├── main.server.dart          # sunucu girişi (static/server)
│   ├── main.client.dart          # istemci girişi
│   ├── app.dart                  # Router + kök yerleşim
│   ├── core/
│   │   ├── theme.dart            # @css global stiller, renk sabitleri
│   │   └── validators.dart       # saf Dart yardımcılar
│   ├── data/
│   │   ├── models/               # @encoder/@decoder'lı modeller
│   │   └── services/             # API istemcileri
│   ├── state/                    # jaspr_riverpod provider'ları
│   ├── components/
│   │   ├── ui/                   # Buton, Kart, Input...
│   │   └── layout/               # Header, Footer, Shell
│   └── pages/                    # route başına bir dosya
├── web/
│   ├── images/  fonts/  styles.css  favicon.ico  robots.txt
├── test/
├── pubspec.yaml
└── Dockerfile                    # server modu için
```

## Öğrenme Roadmap'i (Özet)

1. **Temel:** Dart tazele → JasprPad'de oyna → static proje kur → component'ler ve HTML component fonksiyonları.
2. **Orta:** Routing (multi/single page farkı) → formlar ve event'ler → @css stilleri → state (setState, jaspr_riverpod).
3. **Full-stack:** Scope kavramı → @client/@sync → sunucuda veri çekme → API/backend entegrasyonu.
4. **İleri:** SEO meta yönetimi → kod bölme → custom backend (shelf/dart_frog/Serverpod) → auth.
5. **Uzmanlaşma:** jaspr_test ile test piramidi → CI/CD deploy → kendi component kütüphaneni yazıp pub.dev'e `#jaspr` etiketiyle yükle.

---

[⬅ Ek C — Best Practices](ek-c-best-practices.md) | [Sonraki: Kaynaklar ➡](kaynaklar.md)
