# 18. Gerçek Proje Örnekleri

## 18.1 Proje 1: Basit Blog (static mod)

**Hedef:** Markdown içerikli, SEO dostu, statik bir blog. **Mod:** static.

İçerik ağırlıklı siteler için resmi `jaspr_content` paketi önerilir: markdown dosyalarını yükler, frontmatter'ı ayrıştırır, route'ları otomatik üretir ve hazır blog/dokümantasyon yerleşimleri sunar. Elle kurulum istersen yapı şöyledir:

```text
lib/
├── main.server.dart       # Document(title:..., body: App())
├── main.client.dart       # ClientApp()
├── app.dart               # Router + ShellRoute (header/footer)
├── data/posts.dart        # Yazı listesi (build zamanında yüklenir)
└── pages/
    ├── home_page.dart     # Yazı listesi
    └── post_page.dart     # Tekil yazı
```

**lib/app.dart**

```dart
class App extends StatelessComponent {
  @override
  Component build(BuildContext context) {
    return Router(routes: [
      ShellRoute(
        builder: (context, state, child) => div([
          SiteHeader(),
          child,
          SiteFooter(),
        ]),
        routes: [
          Route(path: '/', builder: (context, state) => HomePage(posts: tumYazilar)),
          for (final yazi in tumYazilar)
            Route(
              path: '/yazilar/${yazi.slug}',
              settings: RouteSettings(changeFreq: ChangeFreq.monthly),
              builder: (context, state) => PostPage(yazi: yazi),
            ),
        ],
      ),
    ]);
  }
}
```

**lib/pages/post_page.dart**

```dart
class PostPage extends StatelessComponent {
  const PostPage({required this.yazi, super.key});
  final BlogYazisi yazi;

  @override
  Component build(BuildContext context) {
    return .fragment([
      // Sayfa özelinde SEO:
      Document.head(
        title: yazi.baslik,
        meta: {'description': yazi.ozet},
      ),
      article([
        h1([.text(yazi.baslik)]),
        p(classes: 'tarih', [.text(yazi.tarih)]),
        RawText(yazi.htmlIcerik), // markdown'dan dönüştürülmüş HTML
      ]),
    ]);
  }
}
```

Build: `jaspr build --sitemap-domain https://blogum.com` → her yazı için ayrı HTML + sitemap üretilir.

## 18.2 Proje 2: REST API Kullanan Dashboard (client mod)

**Hedef:** Login arkası, API'den veri çeken tek sayfalık panel. **Mod:** client (SEO gerekmiyor, sunucu maliyeti sıfır).

```text
lib/
├── main.client.dart
├── app.dart               # Router (single-page)
├── data/api_client.dart   # http paketiyle API istemcisi
├── state/providers.dart   # jaspr_riverpod provider'ları
└── pages/
    ├── dashboard_page.dart
    └── rapor_page.dart    # Route.lazy ile lazy load
```

```dart
// state/providers.dart
final istatistikProvider = FutureProvider<Istatistik>((ref) async {
  return ApiClient().istatistikleriGetir();
});

// pages/dashboard_page.dart
class DashboardPage extends StatelessComponent {
  @override
  Component build(BuildContext context) {
    final istatistik = context.watch(istatistikProvider);
    return istatistik.when(
      loading: () => p([.text('Yükleniyor...')]),
      error: (e, _) => p([.text('Hata: $e')]),
      data: (veri) => div([
        MetrikKarti(baslik: 'Kullanıcı', deger: '${veri.kullaniciSayisi}'),
        MetrikKarti(baslik: 'Satış', deger: '${veri.satisSayisi}'),
      ]),
    );
  }
}
```

## 18.3 Proje 3: Authentication İçeren Uygulama (server mod)

**Hedef:** Bölüm 15'teki kalıpların uçtan uca uygulanması. **Mod:** server + custom backend (shelf).

1. `POST /giris` endpoint'i kimliği doğrular, `HttpOnly` cookie yazar.
2. SSR sırasında her istekte `context.cookies['oturum']` okunur, kullanıcı bir `InheritedComponent` (AuthState) ile ağaca verilir.
3. Router `redirect`'i `/panel/*` yollarını korur.
4. Logout: cookie silinir, `/`'e yönlendirilir.

```dart
void main() async {
  Jaspr.initializeAll(options: defaultServerOptions);

  var handler = serveApp((request, render) {
    if (request.url.path == 'giris' && request.method == 'POST') {
      return girisIsle(request); // cookie yazar
    }
    if (request.url.path == 'cikis') {
      return cikisIsle(request); // cookie siler
    }
    return render(App());
  });

  await shelf_io.serve(handler, InternetAddress.anyIPv4, 8080);
}
```

## 18.4 Proje 4: CRUD Uygulaması (server mod, @sync ile)

**Hedef:** Not listesi: sunucuda yüklenir, istemcide filtrelenir/düzenlenir, değişiklikler API'ye gönderilir.

- `PreloadStateMixin` + `@sync`: notlar sunucuda yüklenir, istemciye taşınır (Bölüm 8.2).
- Yeni not ekleme: form → `fetch` ile kendi endpoint'ine `POST` → başarıda yerel state güncellenir.
- Silme/düzenleme de aynı kalıpla; optimistik UI (önce UI güncelle, hata olursa geri al) kullanıcı deneyimini iyileştirir.

> [!TIP]
> **Final Projesi**
> Dört projeyi birleştir: static blog + admin paneli (server mod, auth'lu, CRUD). Blog yazılarını panelden yönet; public kısım build zamanında statik üretilsin. Bu, Jaspr'ın tüm temel yeteneklerini tek projede kullanmanı sağlar.

---

[⬅ 17. Testing](17-testing.md) | [Sonraki: Ek A — Cheat Sheet ➡](ek-a-cheat-sheet.md)
