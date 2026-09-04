# 0. Başlamadan Önce: Yol Haritası ve Ön Bilgiler

Bu döküman, Dart dilini bilen ve web geliştirmeye **Jaspr** framework'ü ile adım atmak isteyen geliştiriciler için hazırlanmış, aşamalı bir kurs niteliğindedir. Resmi Jaspr dökümantasyonu (`docs.jaspr.site`) esas alınmış; eksik kalan pratik noktalar deneyimsel bilgilerle tamamlanmıştır. Döküman hazırlandığı tarihte güncel ana sürüm **Jaspr 0.23.x**'tir.

## 0.1 Öğrenme Yol Haritası

| Aşama | Konu | Hedef |
|-------|------|-------|
| 1. Hafta | Bölüm 0–3 (Kurulum, temeller, component sistemi) | İlk sayfanı çalıştırmak, HTML component'lerini kullanmak |
| 2. Hafta | Bölüm 4–7 (Dart entegrasyonu, routing, state, formlar) | Çok sayfalı, etkileşimli bir site kurmak |
| 3. Hafta | Bölüm 8–11 (API, rendering modları, mimari, CSS) | Veri çeken, düzgün tasarlanmış bir uygulama |
| 4. Hafta | Bölüm 12–17 (Asset, JS interop, SEO, auth, deploy, test) | Production'a çıkacak kalitede uygulama |
| Sonrası | Bölüm 18 (Gerçek projeler) + Ekler | Portföy projeleri ve referans kullanımı |

## 0.2 Gerekli Ön Bilgiler

- **Dart temelleri (zorunlu):** Değişkenler, fonksiyonlar, sınıflar, null safety, `async/await`. Bunları bilmiyorsan önce `dart.dev`'deki dil turunu tamamla.
- **Temel HTML/CSS bilgisi (zorunlu):** Jaspr, Flutter gibi canvas'a çizim yapmaz; gerçek HTML elementleri ve CSS üretir. `div`, `p`, `class`, flexbox gibi kavramlara aşina olmalısın.
- **Flutter widget sistemi (şiddetle önerilir):** `StatelessWidget`, `StatefulWidget`, `setState`, widget ağacı kavramları. Jaspr bunların neredeyse birebir karşılığını kullanır; resmi dökümantasyon da bu bilgiyi varsayar.
- **HTTP ve REST API mantığı (önerilir):** 8. bölümden itibaren gerekecek.

## 0.3 Önerilen Çalışma Sırası

1. Her bölümü sırayla oku; bölümler birbirinin üzerine inşa edilir.
2. Kod örneklerini kopyala-yapıştır yapma; kendi projende elle yaz. Özellikle 3. bölümden itibaren yanında açık bir `jaspr serve` oturumu olsun.
3. Her bölüm sonundaki **Alıştırma** kutusundaki görevi tamamlamadan bir sonraki bölüme geçme.
4. Takıldığında Ek B'deki "Sık Yapılan Hatalar"a ve resmi dökümantasyona dön.
5. Bölüm 18'deki projeleri sıfırdan, bu rehbere bakmadan yazmaya çalış; sadece takıldığında başvur.

> [!NOTE]
> **JasprPad — Kurulumsuz Deneme**
> Kurulum yapmadan önce Jaspr'ı denemek istersen, DartPad benzeri çevrimiçi editör **JasprPad**'i (playground.jaspr.site) kullanabilirsin. Tarayıcıda örnekleri çalıştırabilir, tutorial'ı takip edebilir ve projeyi Dart projesi olarak indirebilirsin.

> [!NOTE]
> **Sürüm Notu**
> Bu rehber Jaspr 0.23.x ve Dart 3.10+ sürümlerine göre yazılmıştır. `.text()`, `.fragment()` gibi "dot-shorthand" yazımları Dart 3.10 gerektirir. Eski sürümlerle farklar ilgili bölümlerde "Eski Kullanım / Güncel Kullanım" olarak işaretlenmiştir.

---

[⬅ README](../README.md) | [Sonraki: 1. Jaspr Nedir? ➡](01-jaspr-nedir.md)
