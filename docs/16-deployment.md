# 16. Deployment (Yayına Alma)

## 16.1 Build Alma

```bash
jaspr build
```

Çıktı `build/jaspr` klasörüne yazılır:

- **static / client:** Tamamen statik dosyalar — herhangi bir statik hosting'e yüklenebilir.
- **server:** `app` sunucu çalıştırılabilir dosyası + `web/` asset klasörü. İkisini birlikte deploy etmelisin.

## 16.2 Statik Hosting: Firebase Hosting

1. `jaspr build`
2. `firebase init` → "Hosting" seç → public dizin olarak `build/jaspr` gir → static modda "single-page app?" sorusuna **Hayır**, client modda **Evet** → `index.html`'in üzerine yazma sorusuna **Hayır**.
3. `firebase deploy`

## 16.3 GitHub Pages (GitHub Actions ile)

Repo ayarlarında Pages → Source: "GitHub Actions" → "Static HTML" workflow'unu seç ve şu adımları ekle:

```yaml
- name: 'Setup Dart'
  uses: dart-lang/setup-dart@v1.3
- name: 'Build Jaspr'
  run: |
    dart pub global activate jaspr_cli
    jaspr build --verbose
```

"Upload artifact" adımında `path: 'build/jaspr'` yap. Repo adı `<kullanici>.github.io` değilse site `https://<kullanici>.github.io/<repo>` altında yayınlanır; bu durumda base URL'i ayarlamalısın:

```dart
// static modda:
Document(base: '<repo>', body: ...)
```

```html
<!-- client modunda web/index.html içinde: -->
<base href="/<repo>/" />
```

ve tüm linklerin/asset yollarının **göreli** (başında `/` olmayan) olduğundan emin ol. Flutter embedding/plugins kullanıyorsan workflow'a ayrıca `subosito/flutter-action@v2` adımını ekle.

## 16.4 Netlify / Vercel

Resmi dokümantasyonda bu servisler için hazır rehber yoktur; ancak static mod çıktısı standart olduğundan genel kalıp şudur: build komutu olarak `dart pub global activate jaspr_cli && jaspr build` çalıştıracak bir build imajı ayarla (Dart destekleyen) ve yayın dizini olarak `build/jaspr` ver. Vercel/Netlify'nin hazır Dart desteği sınırlı olduğundan en sorunsuz yol GitHub Actions'da build alıp çıktıyı yayınlamaktır.

## 16.5 Docker (Server Modu)

Cloud Run, fly.io, Digital Ocean, AWS gibi container destekleyen her servise deploy edebilirsin. Resmi önerilen Dockerfile:

```dockerfile
# Resmi dart imajını build imajı olarak kullan
FROM dart:stable AS build

# Jaspr CLI'yi aktifleştir
RUN dart pub global activate jaspr_cli

WORKDIR /app
COPY . .

# Bağımlılıkları çöz
RUN rm -f pubspec_overrides.yaml
RUN dart pub get

# Projeyi derle
RUN dart pub global run jaspr_cli:jaspr build --verbose

# Yeni boş imaj — final container
FROM scratch

# Dart runtime kütüphanelerini kopyala
COPY --from=build /runtime/ /
# Site build çıktılarını kopyala
COPY --from=build /app/build/jaspr/ /app/

WORKDIR /app

EXPOSE 8080
CMD ["./app"]
```

```bash
docker build -t sitem .
docker run -p 8080:8080 sitem
```

Flutter embedding/plugins kullanıyorsan build imajını `ghcr.io/cirruslabs/flutter:stable` yap ve runtime kütüphanelerini ayrıca `dart:stable` imajından kopyala (resmi dokümandaki genişletilmiş Dockerfile).

## 16.6 Kendi Sunucun (VPS)

1. Sunucuya Dart SDK kur (veya build'i lokalde/CI'da alıp yalnızca `build/jaspr` içeriğini yükle).
2. `./app` çalıştırılabilir dosyasını bir systemd servisi olarak çalıştır.
3. Önüne Caddy/Nginx koy: TLS sertifikası ve reverse proxy (80/443 → 8080).

## 16.7 Environment Variables

Sunucu tarafında ortam değişkenlerini standart `Platform.environment` (dart:io) ile oku; gizli anahtarları asla istemci koduna koyma — istemciye derlenen her şey herkes tarafından görülebilir. Build argümanları yerine çalışma zamanı değişkenleri kullan (Docker'da `-e ANAHTAR=deger`).

> [!TIP]
> **Alıştırma 16**
> 1. Static bir projeyi GitHub Pages'a Actions ile yayınla.
> 2. Server modu projesi için Dockerfile oluştur, imajı lokalde çalıştır.
> 3. Firebase Hosting'e deploy et; iki modda da "single-page app" sorusuna verilen cevabın farkını açıkla.

---

[⬅ 15. Authentication](15-auth.md) | [Sonraki: 17. Testing ➡](17-testing.md)
