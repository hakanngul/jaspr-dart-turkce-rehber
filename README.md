# Jaspr + Dart — Türkçe Rehber

> Sıfırdan gerçek projeye: kurulumdan deployment'a, component mimarisinden state yönetimine, SSR/SSG'den test stratejilerine kadar **resmi Jaspr dokümantasyonu** temel alınarak hazırlanmış aşamalı Türkçe kurs.

![Jaspr](https://img.shields.io/badge/Jaspr-v0.23.x-1a5a8a)
![Dart](https://img.shields.io/badge/Dart-3.10%2B-0175C2?logo=dart&logoColor=white)
![Dil](https://img.shields.io/badge/Dil-Türkçe-e30a17)
![Bölümler](https://img.shields.io/badge/İçerik-18%20bölüm%20%2B%204%20ek-3f6b46)
![Lisans](https://img.shields.io/badge/Lisans-MIT-green)

Bu doküman, **Dart** dilini bilen ve web geliştirmeye **Jaspr** framework'ü ile adım atmak isteyen geliştiriciler için hazırlanmış, aşamalı bir kurs niteliğindedir. Resmi Jaspr dokümantasyonu (`docs.jaspr.site`) esas alınmış; eksik kalan pratik noktalar deneyimsel bilgilerle tamamlanmıştır. Doküman hazırlandığı tarihte güncel ana sürüm **Jaspr 0.23.x**'tir.

---

## İçindekiler

| # | Bölüm |
|---|-------|
| 0 | [Başlamadan Önce: Yol Haritası ve Ön Bilgiler](docs/00-yol-haritasi.md) |
| 1 | [Jaspr Nedir?](docs/01-jaspr-nedir.md) |
| 2 | [Kurulum ve Başlangıç](docs/02-kurulum.md) |
| 3 | [Jaspr Temelleri: Component Sistemi ve HTML](docs/03-temeller.md) |
| 4 | [Dart ile Jaspr Geliştirme](docs/04-dart-ile-gelistirme.md) |
| 5 | [Routing (Sayfa Yönlendirme)](docs/05-routing.md) |
| 6 | [State Management (Durum Yönetimi)](docs/06-state-management.md) |
| 7 | [Formlar ve Kullanıcı Etkileşimleri](docs/07-formlar.md) |
| 8 | [API ve Backend Entegrasyonu](docs/08-api-backend.md) |
| 9 | [Rendering Modları: SSR, SSG ve Client](docs/09-rendering-modlari.md) |
| 10 | [Component Mimarisi ve Proje Organizasyonu](docs/10-component-mimarisi.md) |
| 11 | [CSS ve UI Geliştirme](docs/11-css-ui.md) |
| 12 | [Asset Kullanımı](docs/12-asset-kullanimi.md) |
| 13 | [JavaScript ile Entegrasyon](docs/13-javascript-entegrasyon.md) |
| 14 | [SEO ve Performans](docs/14-seo-performans.md) |
| 15 | [Authentication ve Authorization](docs/15-auth.md) |
| 16 | [Deployment (Yayına Alma)](docs/16-deployment.md) |
| 17 | [Testing (Test Yazımı)](docs/17-testing.md) |
| 18 | [Gerçek Proje Örnekleri](docs/18-gercek-projeler.md) |
| Ek A | [Cheat Sheet — En Sık Kullanılan Yapılar](docs/ek-a-cheat-sheet.md) |
| Ek B | [Sık Yapılan Hatalar ve Çözümleri](docs/ek-b-sik-hatalar.md) |
| Ek C | [Best Practices ve Proje Checklist'i](docs/ek-c-best-practices.md) |
| Ek D | [Proje Klasör Yapısı ve Öğrenme Roadmap'i](docs/ek-d-klasor-roadmap.md) |
| • | [Kaynaklar](docs/kaynaklar.md) |

---

## Öğrenme Yol Haritası

| Aşama | Konu | Hedef |
|-------|------|-------|
| 1. Hafta | Bölüm 0–3 (Kurulum, temeller, component sistemi) | İlk sayfanı çalıştırmak, HTML component'lerini kullanmak |
| 2. Hafta | Bölüm 4–7 (Dart entegrasyonu, routing, state, formlar) | Çok sayfalı, etkileşimli bir site kurmak |
| 3. Hafta | Bölüm 8–11 (API, rendering modları, mimari, CSS) | Veri çeken, düzgün tasarlanmış bir uygulama |
| 4. Hafta | Bölüm 12–17 (Asset, JS interop, SEO, auth, deploy, test) | Production'a çıkacak kalitede uygulama |
| Sonrası | Bölüm 18 (Gerçek projeler) + Ekler | Portföy projeleri ve referans kullanımı |

## Gerekli Ön Bilgiler

- **Dart temelleri (zorunlu):** Değişkenler, fonksiyonlar, sınıflar, null safety, `async/await`.
- **Temel HTML/CSS bilgisi (zorunlu):** Jaspr gerçek HTML elementleri ve CSS üretir.
- **Flutter widget sistemi (şiddetle önerilir):** `StatelessWidget`, `StatefulWidget`, `setState`, widget ağacı.
- **HTTP ve REST API mantığı (önerilir):** 8. bölümden itibaren gerekir.

## Nasıl Çalışılmalı?

1. Her bölümü sırayla oku; bölümler birbirinin üzerine inşa edilir.
2. Kod örneklerini kopyala-yapıştır yapma; kendi projende elle yaz.
3. Her bölüm sonundaki **Alıştırma** görevini tamamlamadan devam etme.
4. Takıldığında [Ek B — Sık Yapılan Hatalar](docs/ek-b-sik-hatalar.md)'a ve resmi dokümantasyona dön.

---

Bu rehber resmi Jaspr dokümantasyonu (docs.jaspr.site, v0.23.x) temel alınarak hazırlanmıştır. — Eylül 2026
