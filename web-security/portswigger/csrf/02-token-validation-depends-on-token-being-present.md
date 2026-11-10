# CSRF #2 — Token validation depends on token being present

- **Seviye:** Practitioner
- **Sınıf:** CSRF (token bypass)
- **Yöntem:** Rehberli (kavramı derinleştirmek için adım adım birlikte)
- **Araç:** Burp Repeater (deney) + exploit server

---

## Bu lab neyi öğretti?

Dün (CSRF #1) token **hiç yoktu**. Bugün token **var** — yani form "korumalı" görünüyor.
Ama savunma **sahte**: sunucu token'ı yalnızca **varken** doğruluyor; token'ı **tamamen silersen**
kontrol mekanizması devreye girmiyor. "Token var ama koruma yok"un canlı kanıtı.

## Kilit kavrayış 🔑

> **"Token var mı?" yanlış soru. Doğru soru: "Sunucu token YOKKEN ne yapıyor?"**

Bu sunucu, token'ın **yokluğunu** bir saldırı değil, "demek ki gerekmiyor" diye yorumluyor.
Kusur doğrulama **mantığında** — token'ın kendisinde değil.

## Hedef istek (Burp'ten)

```
POST /my-account/change-email HTTP/2
Host: <lab>.web-security-academy.net
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=deneme@deneme.com&csrf=gQZaSeOhCIFAxKwCSJvyYMqjan8CfhcM
```

Dünden fark: body'de artık bir **`csrf`** token var (rastgele, oturuma bağlı, forma gizli gömülü).

## İki deney (Repeater) — savunmasızlığı deneyle kanıtlama

| Deney | Yapılan | Sonuç | Kanıt |
|-------|---------|-------|-------|
| **A** | `csrf` değerinin son harfini değiştir | `Invalid CSRF token` → **reddedildi** | Sunucu token'ı **varken gerçekten doğruluyor** |
| **B** | `&csrf=...` parçasının **tamamını** sil | **Kabul edildi**, e-posta değişti | Token **yokken** sunucu kontrolü **atlıyor** |

Bu iki deney "ne durumda var / ne durumda yok" sorusunu somutlaştırır:
**A = savunma çalışıyor, B = savunma sahte.** Zafiyet B'de yaşıyor.

## Exploit (saldırgan sayfası)

Madem token **yokken** kabul ediliyor, saldırgan sayfası token'ı **hiç koymaz** — yani exploit
dün ile **neredeyse birebir aynı**, sadece `csrf` input'u yok.

```html
<html>
  <body>
    <form action="https://<lab>.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="okey@email.com">
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

> İlginç olan: **aynı saldırı** (token'ı koymama), "korumalı görünen" bir sunucuda da işe yaradı.
> Değişen exploit değil, **sunucunun yanılgısı.**

## Precision dersleri (bu lab'da takıldığım yerler)

- `action` = **`/my-account/change-email`** — baktığım sayfa (`/my-account?id=wiener`) DEĞİL.
  Ders: `action` URL'ini **Burp'teki POST satırından** kopyala, tahminle yazma.
- HTML attribute'ları `=` ile yazılır, `:` ile değil (`action="..."`, `action:"..."` değil).
- Şema tek: `https://`, `https://https://` değil.
- Gönderilecek veri ayrı bir `<input>` elementidir, `<form>` etiketine tıkılmaz.

## Remediation (defender tarafı)

1. **Token'ı HER istekte zorunlu kıl:** Token yoksa isteği **reddet** (eksikliği "muaf" sayma).
   Kusurun tek cümlelik özeti: doğrulama "token varsa kontrol et" değil, **"token yoksa REDDET"**
   olmalı.
2. **Token geçerliliği + oturum bağı:** Token hem doğru hem de kullanıcının oturumuna ait olmalı.
3. **SameSite cookie** (`Lax`/`Strict`): İkincil savunma; cross-site isteğe cookie'yi göndermez.
4. **Framework'ün hazır CSRF korumasını kullan** ve "token opsiyonel" moduna asla düşürme.

## Zincir bağı

- **CSRF #1 (no defenses):** token yoktu → en basit.
- **CSRF #2 (bu):** token var **ama doğrulama koşullu** → "savunma teatral" ailesi.
- Genel ilke: Bir güvenlik kontrolü **koşullu** uygulanıyorsa (yalnızca X durumunda), saldırgan
  **X'i bozarak** kontrolü atlatır. Burada X = "token mevcut". Aynı desen başka bypass'larda da:
  "yalnızca POST'ta kontrol" → GET'e çevir; "yalnızca belli Content-Type'ta" → tipi değiştir.

## Köşe taşı

> Bir savunma yalnızca belli bir koşulda devreye giriyorsa, gerçek güvenlik o koşulu
> **saldırganın bozamayacağını** garanti etmekten geçer. "Token varsa doğrula" yetmez;
> "token yoksa reddet" gerekir.
