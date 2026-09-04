# 17. Testing (Test Yazımı)

## 17.1 jaspr_test Paketi

Jaspr'ın kendi test paketi `jaspr_test`, `package:test` üzerine kuruludur ve API'si `flutter_test`'e benzer:

```bash
dart pub add jaspr_test --dev
```

## 17.2 Üç Test Fonksiyonu

Full-stack bir framework olduğu için üç test ortamı vardır:

| Fonksiyon | Ortam | Kullanım |
|-----------|-------|----------|
| `testComponents` | Simüle edilmiş ortam | Component birim testleri (Flutter widget testi gibi) |
| `testClient` | Headless tarayıcı | DOM etkileşimleri; `url` ve `initialStateData` ile senkron state simülasyonu |
| `testServer` | Sanal HTTP sunucusu | SSR çıktısının istek/yanıt düzeyinde testi |

## 17.3 Component Testi Örneği

**test/sayac_test.dart**

```dart
// package:test'i de export eder; ek import gerekmez
import 'package:jaspr_test/jaspr_test.dart';

import '../lib/components/sayac.dart';

void main() {
  group('Sayac component testi', () {
    testComponents('sayac artirilabilmeli', (tester) async {
      // Component'i test ortamına yükle:
      tester.pumpComponent(Sayac());

      // 'Sayaç: 0' metni render edilmiş olmalı:
      expect(find.text('Sayaç: 0'), findsOneComponent);

      // <button> elementini bul ve tıklamayı simüle et:
      await tester.click(find.tag('button'));

      // 'Sayaç: 1' metni görünmeli:
      expect(find.text('Sayaç: 1'), findsOneComponent);
    });
  });
}
```

Testleri çalıştır:

```bash
dart test
```

## 17.4 Bulucular (Finders)

- `find.text('...')` — metin içeriğine göre
- `find.tag('button')` — HTML etiketine göre
- Ayrıca component tipine, key'e ve CSS benzeri seçicilere göre bulucular mevcuttur.

## 17.5 Test Stratejisi

1. **Saf mantık → package:test:** Validators, servisler, modeller. Component'siz, hızlı birim testleri.
2. **Component'ler → testComponents:** Her kritik component için render + etkileşim testi.
3. **SSR davranışı → testServer:** Doğru durum kodu, meta etiketler, pre-render içeriği.
4. **Kritik akışlar → testClient:** Form gönderimi, navigasyon gibi gerçek DOM etkileşimleri.

> [!TIP]
> **Alıştırma 17**
> 1. Bölüm 7'deki `Validators` sınıfına birim testleri yaz.
> 2. Sayaç component'i için `testComponents` ile artır/azalt testleri ekle.
> 3. Server modunda 404 sayfasının doğru durum kodu döndürdüğünü `testServer` ile doğrula.

---

[⬅ 16. Deployment](16-deployment.md) | [Sonraki: 18. Gerçek Proje Örnekleri ➡](18-gercek-projeler.md)
