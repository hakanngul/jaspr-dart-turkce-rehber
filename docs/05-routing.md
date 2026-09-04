# 5. Routing (Sayfa Yönlendirme)

## 5.1 Temel Kurulum

Routing için çekirdek `jaspr_router` paketi kullanılır. API'si Flutter'daki `go_router`'a çok benzer; go_router biliyorsan kavramlar tanıdık gelecektir.

```bash
dart pub add jaspr_router
```

**lib/app.dart**

```dart
import 'package:jaspr/jaspr.dart';
import 'package:jaspr_router/jaspr_router.dart';

import 'pages/home.dart';
import 'pages/about.dart';

class App extends StatelessComponent {
  const App({super.key});

  @override
  Component build(BuildContext context) {
    return Router(routes: [
      Route(path: '/', builder: (context, state) => Home()),
      Route(path: '/about', builder: (context, state) => About()),
    ]);
  }
}
```

Her `Route` bir `path` şablonu ve o URL eşleştiğinde render edilecek component'i döndüren bir `builder` fonksiyonu alır.

## 5.2 Multi-Page vs Single-Page Routing

Yeni proje oluştururken iki routing stratejisinden birini seçersin:

|  | Multi-page (sunucu) | Single-page (istemci) |
|--|---------------------|------------------------|
| Navigasyon | Gerçek sayfa yüklemesi; tarayıcı yeni sayfayı sunucudan ister | Tamamen istemcide; sunucuya istek yok |
| Koşul | Router, ağacın yalnızca sunucuda pre-render edilen bölümünde (örn. `@client` component'lerin üstünde) | Router, istemcide render edilen bölümde (örn. `@client` component'lerin altında) |
| Mod gereksinimi | static veya server | Tüm modlar |
| Navigasyon API | Normal linkler (`Link` / `<a>`) | `context.push(...)` vb. |
| Tipik kullanım | Geleneksel çok sayfalı siteler, blog, kurumsal site | Uygulama benzeri siteler, dashboard |

> [!WARNING]
> **Yaygın Hata**
> Multi-page kurulumda (Router yalnızca sunucuda) `Router.of(context).push('/yol')` çağırmak çalışmaz — istemcideki component ağacında Router bulunamaz. Bu durumda navigasyon için `Link` component'ini kullan.

## 5.3 Dinamik Route'lar ve URL Parametreleri

Yol segmentinin başına `:` koyarak parametre tanımlarsın; değere `RouteState` üzerinden erişirsin:

```dart
Route(
  path: '/users/:userId',
  builder: (context, state) => UserScreen(id: state.params['userId']!),
),
```

Query parametreleri (`?` sonrası) için `state.queryParams` kullanılır — `/users?filter=admins` örneği:

```dart
Route(
  path: '/users',
  builder: (context, state) => UsersScreen(filter: state.queryParams['filter']),
),
```

## 5.4 Nested (İç İçe) Route'lar

```dart
Route(
  path: '/users',
  builder: (context, state) => const UsersScreen(),
  routes: [
    Route(
      path: ':userId',
      builder: (context, state) => UserScreen(id: state.params['userId']!),
    ),
  ],
),
```

`/users` → `UsersScreen`; `/users/abc` → `UserScreen` render edilir. Flutter'daki GoRouter'dan farklı olarak iç içe route'lar sayfa yığını oluşturmaz; yalnızca en içteki eşleşen route render edilir.

### ShellRoute — Ortak Yerleşim

Navigasyon çubuğu gibi her sayfada sabit kalan bir yerleşim için `ShellRoute` kullanılır:

```dart
ShellRoute(
  builder: (context, state, child) {
    return div([
      NavigasyonCubugu(), // her sayfada görünür
      child,              // aktif sayfanın içeriği
    ]);
  },
  routes: [
    Route(path: '/details', builder: (context, state) => const DetailsScreen()),
  ],
),
```

## 5.5 Navigation

En basit yol `Link` component'idir — `<a>` etiketinin akıllı karşılığıdır; tıklandığında kurulumuna uygun doğru navigasyonu yapar:

```dart
Link(href: '/about', [.text('Hakkında')])
```

Programatik navigasyon (yalnızca single-page routing'de):

```dart
context.push('/users/123');                 // yeni geçmiş kaydı ekler
context.replace('/users/123');              // mevcut kaydı değiştirir
context.back();                             // tarayıcı geri tuşu
context.pushNamed('users', params: {'userId': '123'}); // isimli route
```

İsimli route tanımı:

```dart
Route(
  name: 'users',
  path: '/users/:userId',
  builder: (context, state) => UserScreen(id: state.params['userId']!),
),
```

Navigasyonla ek veri taşımak (`extra` yalnızca String, bool, int, double ve bunların List/Map'leri olabilir — tarayıcıda serialize edilir):

```dart
context.push('/123', extra: 'abc');
// Hedefte okuma:
final veri = RouteState.of(context).extra! as String;
```

## 5.6 Redirect ve 404

Yönlendirme, örneğin giriş yapmamış kullanıcıyı login sayfasına göndermek için kullanılır. `null` döndürmek "yönlendirme yok" demektir:

```dart
Router(
  redirect: (context, state) {
    if (!AuthState.of(context).isSignedIn) {
      return '/signin'; // giriş yoksa login'e gönder
    }
    return null; // izin ver
  },
  routes: [ /* ... */ ],
)
```

Üst seviye (`Router`'da) ve route seviyesi (`Route`'ta) olmak üzere iki tür redirect vardır. Eşleşmeyen URL'ler için Router'ın `errorBuilder` parametresiyle kendi 404 sayfanı tanımlayabilirsin:

```dart
Router(
  routes: [ /* ... */ ],
  errorBuilder: (context, state) => div([
    h1([.text('404')]),
    p([.text('Sayfa bulunamadı.')]),
    Link(href: '/', [.text('Ana sayfaya dön')]),
  ]),
)
```

## 5.7 Lazy Routes (Kod Bölme)

Büyük uygulamalarda tüm sayfaları tek seferde yüklemek yerine, route'a özel JS parçalarını gerektiğinde yüklersin. Dart'ın `deferred` import'u ile çalışır:

```dart
import 'pages/home.dart';
import 'pages/about.dart' deferred as about;

Router(
  routes: [
    Route(path: '/', builder: (context, state) => Home()),
    Route.lazy(
      path: '/about',
      builder: (context, state) => about.About(),
      load: about.loadLibrary,
    ),
  ],
)
```

`/about`'a gidildiğinde ilgili JS dosyası o anda indirilir. Ayrıca `Router.of(context).preload('/about')` ile kullanıcı linke gelmeden (örn. fare üzerine gelince) önceden yükleyebilirsin; `Link` component'inde bu davranış `preload` parametresiyle hazır gelir.

> [!TIP]
> **Alıştırma 5**
> 1. Üç sayfalı bir site kur: Ana Sayfa, Hakkında, İletişim. `ShellRoute` ile hepsinde ortak bir menü göster.
> 2. `/urunler/:id` dinamik route'u ekle; `id` parametresini sayfada göster.
> 3. Bir 404 sayfası tanımla ve var olmayan bir URL'yi tarayıcıda açarak test et.
> 4. Bir route'u `Route.lazy`'ye çevir; tarayıcının Network sekmesinde sayfa geçişinde yeni JS parçasının yüklendiğini gözlemle.

---

[⬅ 4. Dart ile Geliştirme](04-dart-ile-gelistirme.md) | [Sonraki: 6. State Management ➡](06-state-management.md)
