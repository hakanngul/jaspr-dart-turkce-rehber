# 12. Asset Kullanımı

## 12.1 Temel Kural: web/ Klasörü

Statik dosyalar (resimler, fontlar, favicon, robots.txt...) projenin `web/` klasörüne konur ve kök URL'den sunulur: `web/images/logo.png` dosyası `/images/logo.png` adresinden erişilebilir. Sunucu giriş noktasındaki `runApp`, `/web` dizinindeki dosyaları otomatik sunar.

```text
web/
├── images/logo.png      →  /images/logo.png
├── styles.css           →  /styles.css
├── favicon.ico          →  /favicon.ico
└── fonts/OzelFont.woff2 →  /fonts/OzelFont.woff2
```

## 12.2 Resim ve SVG

```dart
img(src: '/images/logo.png', alt: 'Site logosu')
```

- **alt** attribute'unu her zaman doldur — SEO ve erişilebilirlik için zorunludur.
- Büyük resimleri build öncesi optimize et; mümkünse `.webp` kullan.
- SVG'yi dosya olarak kullan: `img(src: '/images/ikon.svg')`. SVG'nin içeriğini DOM'a gömmek istersen `RawText()` ile ham HTML basabilirsin (güvenmediğin içerikte XSS riskine dikkat).

## 12.3 Fontlar

```dart
// Document head içine:
Document.head(children: [
  link(rel: 'stylesheet', href: 'https://fonts.googleapis.com/css2?family=Inter&display=swap'),
])
// veya kendi font dosyan web/fonts altındaysa @font-face kuralını
// bir .css dosyasında tanımlayıp link ile dahil et.
```

## 12.4 Static/Server Modunda Preload

Kritik asset'leri `<link rel="preload">` ile önceden yükletebilirsin (Bölüm 14).

> [!TIP]
> **Alıştırma 12**
> 1. Projene bir logo ve bir özel font ekle; başlıkta kullan.
> 2. Bir resmi hem PNG hem WebP olarak ekle, boyutlarını karşılaştır.

---

[⬅ 11. CSS ve UI](11-css-ui.md) | [Sonraki: 13. JavaScript Entegrasyonu ➡](13-javascript-entegrasyon.md)
