# 8. API ve Backend Entegrasyonu

## 8.1 İstemcide REST API Kullanımı

İstemci tarafında `package:http` (tarayıcıda fetch tabanlı çalışır) kullanılır:

```bash
dart pub add http
```

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Post {
  final int id;
  final String baslik;
  Post({required this.id, required this.baslik});

  factory Post.fromJson(Map<String, dynamic> json) => Post(
        id: json['id'] as int,
        baslik: json['title'] as String,
      );
}

Future<List<Post>> postlariGetir() async {
  final res = await http.get(Uri.parse('https://jsonplaceholder.typicode.com/posts'));
  if (res.statusCode == 200) {
    final liste = jsonDecode(res.body) as List;
    return liste.map((j) => Post.fromJson(j)).toList();
  }
  throw Exception('Yükleme başarısız: ${res.statusCode}');
}
```

### API Verisini UI'da Gösterme

```dart
class PostListesi extends StatefulComponent {
  const PostListesi({super.key});
  @override
  State<PostListesi> createState() => _PostListesiState();
}

class _PostListesiState extends State<PostListesi> {
  List<Post>? postlar;
  String? hata;

  @override
  void initState() {
    super.initState();
    postlariGetir().then((veri) {
      setState(() => postlar = veri);
    }).catchError((e) {
      setState(() => hata = e.toString());
    });
  }

  @override
  Component build(BuildContext context) {
    if (hata != null) return p(classes: 'hata', [.text('Hata: $hata')]);
    if (postlar == null) return p([.text('Yükleniyor...')]);
    return ul([
      for (final p in postlar!) li([.text(p.baslik)]),
    ]);
  }
}
```

Üç durum deseni (loading / error / data) her veri yükleme ekranında karşına çıkar; bu deseni bir alışkanlık haline getir.

## 8.2 Sunucuda Veri Çekme (Data Fetching)

> [!NOTE]
> Bu bölüm static ve server modu içindir.

Sunucuda pre-render yaparken veritabanından veya API'den veri çekmek yaygındır. Üç sorun çözülmelidir: (1) pre-render veri gelene kadar bekletilmeli, (2) sunucu kodu istemci derlemesini bozmamalı, (3) yüklenen veri istemciye de aktarılmalı. Jaspr üçünü de çözer.

### AsyncBuilder ve AsyncStatelessComponent

Build metodunda `await` kullanmanı sağlayan, **yalnızca sunucuda** çalışan özel component'lerdir (`package:jaspr/server.dart` import'u gerekir):

```dart
// AsyncBuilder:
return AsyncBuilder(builder: (context) async {
  var veri = await veritabanindanYukle();
  return BaskaComponent(veri: veri);
});

// AsyncStatelessComponent:
class MakalelerSayfasi extends AsyncStatelessComponent {
  @override
  Future<Component> build(BuildContext context) async {
    var makaleler = await makaleleriYukle();
    return MakaleListesi(makaleler: makaleler);
  }
}
```

> [!WARNING]
> **Flutter geliştiricisine not**
> Build içinde `await` kullanmak Flutter'da tehlikelidir ama burada güvenlidir: bu component'ler yalnızca sunucuda ve pre-render sırasında **bir kez** çalışır; pahalı rebuild riski yoktur.

### PreloadStateMixin

`StatefulComponent` ile asenkron veri yüklemek için:

```dart
class SayfamState extends State<Sayfam> with PreloadStateMixin {
  @override
  Future<void> preloadState() async {
    // Yalnızca sunucuda çalışır; initState() ve build'i geciktirir.
    veri = await yukle();
  }
}
```

### Veriyi İstemciye Taşıma

Sunucuda yüklenen veriyi istemciye aktarmak için Bölüm 6.5'teki `@sync` veya `@client` parametrelerini kullan. `@sync` + `PreloadStateMixin` kombinasyonu tipik blog senaryosudur:

```dart
class MakaleListesiState extends State<MakaleListesi>
    with MakaleListesiStateSyncMixin, PreloadStateMixin {
  @sync
  List<String> makaleler = [];

  @override
  Future<void> preloadState() async {
    makaleler = await makaleleriYukle(); // sunucuda çalışır
  }
  // İstemcide makaleler zaten dolu gelir; filtreleme vs. burada yapılır.
}
```

## 8.3 Kendi Backend'ini Kurma (Custom Backend)

Server modunda Jaspr kendi HTTP sunucusunu (shelf tabanlı) çalıştırır. Kendi backend'ini kullanmak istersen iki fonksiyon devreye girer:

- `serveApp(handler)`: Jaspr'ın sunucu tarafını tek bir shelf handler'a dönüştürür.
- `renderComponent(component)`: Bir component'i doğrudan HTML string'e çevirir.

```dart
void main() async {
  var handler = serveApp((request, render) {
    print('İstek: ${request.requestedUri}');
    return render(App());
  });

  await shelf_io.serve(handler, InternetAddress.anyIPv4, 8080);
}
```

Bu sayede **shelf**, **Serverpod** (resmi entegrasyon paketi var) veya **dart_frog** gibi Dart backend framework'leriyle birleştirebilirsin. Örneğin dart_frog route'u:

```dart
Future<Response> onRequest(RequestContext context) {
  return renderJasprComponent(context, MyComponent());
}
```

> [!WARNING]
> **Dikkat**
> Kendi backend'ini kullanınca `jaspr serve` çalışmaya devam eder ama sunucuda otomatik reload çalışmaz. Ayrıca `serveApp` handler'ını `/` dışında bir öneke bağlarsan, `<base href="/on ek/">` etiketini eklemeyi unutma; yoksa statik dosyalar yüklenemez.

## 8.4 Sunucu API'leri: Request ve Response

Server-side rendering sırasında isteğe ait bilgilere `BuildContext` uzantılarıyla erişirsin:

| API | Görevi |
|-----|--------|
| `context.url` | Mevcut isteğin URL'i |
| `context.headers` | İstek header'ları (büyük/küçük harf duyarsız) |
| `context.cookies` | İstek cookie'leri |
| `context.setHeader(ad, deger)` | Yanıta header ekler |
| `context.setCookie(ad, deger, {...})` | Yanıta cookie ekler |
| `context.setStatusCode(404, {...})` | HTTP durum kodunu değiştirir |

> [!WARNING]
> **Yaygın Hata**
> Bu API'lere istemcide de render edilen bir component'ten erişirsen hata alırsın. `if (!kIsWeb) { ... }` kontrolü ve koşullu import gerekir (Bölüm 9.5).

> [!TIP]
> **Alıştırma 8**
> 1. `jsonplaceholder.typicode.com`'dan kullanıcı listesi çekip tablo halinde gösteren bir sayfa yaz (loading/error/data durumlarıyla).
> 2. Static modda `AsyncBuilder` ile build zamanında bir API'den veri çek; üretilen HTML'de verinin gömülü olduğunu doğrula.
> 3. `@sync` + `PreloadStateMixin` ile sunucuda yüklenen listeyi istemcide arama kutusuyla filtrelenebilir yap.

---

[⬅ 7. Formlar](07-formlar.md) | [Sonraki: 9. Rendering Modları ➡](09-rendering-modlari.md)
