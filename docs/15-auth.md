# 15. Authentication ve Authorization

> [!NOTE]
> **Kapsam Notu**
> Resmi Jaspr dokümantasyonunda hazır bir "auth modülü" yoktur; bu bölüm, Jaspr'ın belgelenen yapı taşlarının (cookie'ler, redirect'ler, sunucu context'i) standart web kalıplarıyla nasıl birleştirileceğini gösterir. Bu bölümdeki mimari **server modu** gerektirir.

## 15.1 Temel Mimari

Klasik ve güvenli yaklaşım **oturum çerezi (session cookie)** kullanmaktır:

1. Kullanıcı login formunu gönderir → sunucu kimliği doğrular.
2. Sunucu, `HttpOnly` bir cookie ile oturum kimliği yazar.
3. Sonraki her istekte sunucu, SSR sırasında cookie'yi okuyup kullanıcıyı tanır.
4. Korumalı sayfalar Router `redirect`'i ile korunur.

## 15.2 Cookie Okuma/Yazma

SSR sırasında `BuildContext` uzantılarını kullanırsın (yalnızca sunucuda):

```dart
class ProfilSayfasi extends StatelessComponent {
  @override
  Component build(BuildContext context) {
    // İstek cookie'sini oku (sunucu tarafı):
    final oturumId = context.cookies['oturum'];

    if (oturumId == null) {
      // Yetkisiz erişim: durum kodu ayarla
      context.setStatusCode(401);
      return p([.text('Giriş yapmalısınız.')]);
    }
    return p([.text('Hoş geldin!')]);
  }
}
```

Login işleminde (kendi backend endpoint'inde) cookie yaz:

```dart
// serveApp handler'ı veya context.setCookie ile:
context.setCookie('oturum', yeniOturumId /* , path: '/', httpOnly: true, ... */);
```

## 15.3 Protected Routes (Korumalı Sayfalar)

Router seviyesinde redirect ile koruma (Bölüm 5.6):

```dart
Router(
  redirect: (context, state) {
    final girisliMi = AuthState.of(context).isSignedIn;
    final korumaliYol = state.path.startsWith('/panel');

    if (korumaliYol && !girisliMi) {
      return '/giris'; // panele girmek isteyen girişsiz kullanıcıyı yönlendir
    }
    return null;
  },
  routes: [
    Route(path: '/giris', builder: (context, state) => GirisSayfasi()),
    Route(path: '/panel', builder: (context, state) => PanelSayfasi()),
    Route(path: '/panel/ayarlar', builder: (context, state) => AyarlarSayfasi()),
  ],
)
```

## 15.4 Rol Tabanlı Erişim

Kullanıcı modeline `rol` alanı ekleyip redirect'te kontrol et:

```dart
redirect: (context, state) {
  final kullanici = AuthState.of(context).kullanici;
  if (state.path.startsWith('/admin') && kullanici?.rol != 'admin') {
    return '/yetkisiz';
  }
  return null;
},
```

## 15.5 Token Kullanımı (API senaryosu)

Ayrı bir API sunucun varsa (ör. dart_frog/Serverpod), istemci tarafında JWT benzeri token taşıyabilirsin. Dikkat edilecekler:

- Token'ı `localStorage`'da saklamak XSS'e açıktır; mümkünse `HttpOnly` cookie tercih et.
- SSR'da token'a ihtiyaç varsa cookie zorunludur — `localStorage` sunucuda okunamaz.
- İstemciden API çağrılarında token'ı `Authorization: Bearer ...` header'ıyla gönder.

> [!WARNING]
> **Güvenlik Uyarıları**
> Şifreleri asla düz metin saklama; sunucuda `bcrypt`/`argon2` ile hash'le. Cookie'leri `HttpOnly` + `Secure` + `SameSite` ile ayarla. Auth mantığını istemcide değil her zaman sunucuda doğrula — istemci kontrolleri yalnızca kullanıcı deneyimi içindir.

> [!TIP]
> **Alıştırma 15**
> 1. Sahte bir kullanıcı tablosuyla (Map) login endpoint'i kur; doğru şifrede cookie yaz.
> 2. `/panel` altındaki iki sayfayı redirect ile koru; girişsiz erişimi test et.
> 3. Logout butonu yap: cookie'yi silip ana sayfaya yönlendirsin.

---

[⬅ 14. SEO ve Performans](14-seo-performans.md) | [Sonraki: 16. Deployment ➡](16-deployment.md)
