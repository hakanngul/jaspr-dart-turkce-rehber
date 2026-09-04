# 7. Formlar ve Kullanıcı Etkileşimleri

## 7.1 Event Handling Temelleri

Tüm HTML component'leri `events` parametresiyle DOM event'lerini dinler. Bazı component'lerde (`button`, `input`, `select`, `textarea`) kısayol parametreleri de vardır:

```dart
// Kısayol parametresi:
button(onClick: () => print('Tıklandı'), [.text('Tıkla')])

// Standart events parametresi (Event nesnesine erişim verir):
div(events: {'click': (event) => print('Tıklandı')}, [.text('Tıkla')])

// events() yardımcısı — tip güvenli:
div(events: events(onClick: () => print('Tıklandı')), [.text('Tıkla')])
```

### Input Event'lerinde Tip Güvenliği

`events()` yardımcısının `onInput`/`onChange` parametreleri hedef elemente göre tiplenir:

| Element | Değer tipi |
|---------|------------|
| checkbox / radio input | `bool?` |
| number input | `num?` |
| date input | `DateTime` |
| file input | `List<File>?` |
| select | `List<String>` |
| text input / textarea | `String` |

```dart
input(
  type: InputType.checkbox,
  events: events(onInput: (bool? deger) => print('İşaretli: $deger')),
  [],
)

textarea(
  events: events(onChange: (String deger) => print('İçerik: $deger')),
  [],
)
```

> [!NOTE]
> **onInput vs onChange**
> `onInput` her tuş vuruşunda tetiklenir (anlık arama için ideal); `onChange` alan odağı kaybedince tetiklenir (form değeri kesinleşince çalıştırmak için ideal). Tarayıcı davranışının aynısıdır.

## 7.2 Kontrollü Form Component'i

Jaspr'da form state'i tipik olarak `StatefulComponent` içinde tutulur:

```dart
class GirisFormu extends StatefulComponent {
  const GirisFormu({super.key});
  @override
  State<GirisFormu> createState() => _GirisFormuState();
}

class _GirisFormuState extends State<GirisFormu> {
  String eposta = '';
  String sifre = '';
  String? hata;

  void gonder() {
    // Basit validasyon
    if (!eposta.contains('@')) {
      setState(() => hata = 'Geçerli bir e-posta girin');
      return;
    }
    if (sifre.length < 6) {
      setState(() => hata = 'Şifre en az 6 karakter olmalı');
      return;
    }
    setState(() => hata = null);
    // API'ye gönder...
    print('Giriş: $eposta');
  }

  @override
  Component build(BuildContext context) {
    return form(
      events: {'submit': (e) => e.preventDefault()}, // sayfa yenilemeyi engelle
      [
        input(
          type: InputType.email,
          attributes: {'placeholder': 'E-posta'},
          onInput: (deger) => eposta = deger,
          [],
        ),
        input(
          type: InputType.password,
          attributes: {'placeholder': 'Şifre'},
          onInput: (deger) => sifre = deger,
          [],
        ),
        if (hata != null) p(classes: 'hata', [.text(hata!)]),
        button(onClick: gonder, [.text('Giriş Yap')]),
      ],
    );
  }
}
```

Satır satır kritik noktalar:

- `e.preventDefault()`: HTML form'unun varsayılan "sayfayı yenileyerek submit etme" davranışını engeller. Bunu unutmak en yaygın form hatasıdır.
- `onInput: (deger) => eposta = deger`: Her tuş vuruşunda alan state'i güncellenir; `setState` çağırmıyoruz çünkü bu örnekte UI anında değişmek zorunda değil.
- `gonder()`: Validasyon + gönderim tek yerde; hata varsa state'e yazılır ve UI koşullu olarak gösterir.

## 7.3 Validasyon

Validasyon mantığını form component'inden ayırıp test edilebilir hale getirmek iyi bir pratiktir:

```dart
class Validators {
  static String? eposta(String deger) {
    if (deger.isEmpty) return 'E-posta boş olamaz';
    if (!RegExp(r'^[^@]+@[^@]+\.[^@]+$').hasMatch(deger)) {
      return 'Geçersiz e-posta formatı';
    }
    return null; // null = geçerli
  }

  static String? sifre(String deger) {
    if (deger.length < 8) return 'En az 8 karakter';
    if (!deger.contains(RegExp(r'[0-9]'))) return 'En az bir rakam içermeli';
    return null;
  }
}
```

Bu fonksiyonlar saf Dart olduğu için birim testi yazmak çok kolaydır (Bölüm 17).

## 7.4 Ne Zaman Hangi Yaklaşım?

- Tek alanlı basit arama → `onInput` + yerel state yeterli.
- Çok alanlı form → Validators benzeri ayrık validasyon + tek `gonder()` metodu.
- Çok adımlı sihirbaz formu → Riverpod ile adımlar arası paylaşılan form state'i.
- Sunucu tarafında işlenmesi gereken kritik formlar → server modu + kendi endpoint'in (Bölüm 8).

> [!TIP]
> **Alıştırma 7**
> 1. Kayıt formu yaz: ad, e-posta, şifre, şifre tekrar. Tüm alanları `Validators` benzeri fonksiyonlarla doğrula, hataları alanların altında göster.
> 2. Canlı arama kutusu yap: kullanıcı yazdıkça (`onInput`) bir listede filtreleme yapsın.
> 3. Form gönderilirken "sayfa yenileniyor" sorununu bilerek tetikle (preventDefault'u kaldır), sonra düzelt — farkı gözlemle.

---

[⬅ 6. State Management](06-state-management.md) | [Sonraki: 8. API ve Backend ➡](08-api-backend.md)
