# 13. JavaScript ile Entegrasyon

## 13.1 Temel Kural: universal_web

Tarayıcı API'lerine (`window`, `document`...) erişmek için eski `dart:html` yerine `package:universal_web` kullan. Bu paket hem istemcide gerçek API'yi, hem sunucuda mock'ları sağlar — koşullu import yazmana gerek kalmaz. Yine de çağrıları `kIsWeb` kontrolüyle sar; yoksa sunucu render'ında istisna alırsın:

```dart
import 'package:universal_web/web.dart' as web;

void boyutYazdir() {
  if (kIsWeb) {
    print('Pencere: ${web.window.innerWidth}x${web.window.innerHeight}');
  }
}
```

## 13.2 DOM Elemanına Erişim: GlobalNodeKey

```dart
final GlobalNodeKey<web.HTMLInputElement> inputKey = GlobalNodeKey();

// build içinde:
input(key: inputKey, [])

// sonra, ör. buton tıklamasında:
void odaklan() {
  inputKey.currentNode?.focus();
}
```

## 13.3 Global Event Dinleme

```dart
StreamSubscription? sub;

@override
void initState() {
  super.initState();
  if (kIsWeb) {
    sub = web.EventStreamProviders.resizeEvent.forTarget(web.window).listen((e) {
      print('Pencere yeniden boyutlandı');
    });
  }
}

@override
void dispose() {
  sub?.cancel();
  super.dispose();
}
```

## 13.4 JS Interop — Dart ↔ JavaScript

Jaspr JavaScript'e (veya deneysel olarak `--experimental-wasm` ile WebAssembly'e) derlendiği için mevcut JS kütüphaneleriyle konuşabilirsin. `package:universal_web/js_interop.dart` üzerinden `js_interop`'u güvenle import et:

```dart
import 'package:universal_web/js_interop.dart';

// Üst düzey JS fonksiyonuna erişim:
@JS()
external void alert(JSString message);

void uyariGoster() {
  alert('Dart\'tan merhaba!'.toJS);
}

// JS nesnesinin üyelerine extension type ile erişim:
extension type Console._(JSObject _) implements JSObject {
  external void log(JSAny? item);
}

@JS()
external Console get console;

void konsolaYaz() {
  console.log('Dart\'tan merhaba!'.toJS);
}
```

> [!NOTE]
> **Kural**
> Standart web API'leri için her zaman önce `package:web`/`package:universal_web` kullan; `js_interop`'u yalnızca harici JS kütüphaneleri ve ileri senaryolar için ayır.

## 13.5 Harici JS Dosyası Dahil Etme

Üçüncü parti script'i `Document.head` (server/static) veya `web/index.html` (client) üzerinden ekle:

```dart
Document.head(children: [
  script(src: 'https://cdn.ornek.com/kutuphane.js', defer: true),
])
```

## 13.6 Flutter Embedding (Bonus)

Mevcut bir Flutter Web uygulamanı Jaspr sitesinin içine gömebilirsin (`pubspec.yaml`'a `jaspr: { flutter: embedded }`, ayrıca `build_web_compilers: ^4.4.6` bağımlılığı ve Flutter SDK gerekir). Web destekli Flutter plugin'leri (`shared_preferences`, `firebase`...) için `flutter: plugins` kullanılır. Not: 0.22'den itibaren eski `jaspr_web_compilers` fork'u kaldırıldı.

> [!TIP]
> **Alıştırma 13**
> 1. Pencere genişliğini ekranda canlı gösteren bir component yaz (resize event + setState).
> 2. `GlobalNodeKey` ile "odaklan" butonu yap: tıklayınca input'a focus versin.
> 3. `@JS()` ile tarayıcının `console.log`'una eriş ve bir butondan tetikle.

---

[⬅ 12. Asset Kullanımı](12-asset-kullanimi.md) | [Sonraki: 14. SEO ve Performans ➡](14-seo-performans.md)
