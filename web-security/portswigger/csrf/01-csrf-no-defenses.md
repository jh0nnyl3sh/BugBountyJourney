# CSRF #1 — CSRF vulnerability with no defenses

- **Seviye:** Apprentice
- **Sınıf:** CSRF (Cross-Site Request Forgery) — yeni Tier-1 sınıf
- **Yöntem:** Körleme (link verildi, exploit HTML'i elle kuruldu)
- **Araç:** Burp (isteği incelemek için) + exploit server

---

## Zafiyet nedir?

CSRF, saldırganın **kurbanın oturumunu (session) kötüye kullanarak** onun adına istem dışı bir
işlem yaptırmasıdır. Saldırgan bir şey enjekte etmez, bir şey okumaz — **kurbanın kimliğini silah
yapar.** Niyet saldırganın, ama sunucuya giden "imza" (session cookie) kurbanındır.

## Analoji

Okul, "çocuğumu bugün komşu alsın" diyen **imzalı notu** getireni sorgulamaz; sadece **imza gerçek
mi** diye bakar, notu kimin yazdırdığına değil. Saldırgan, senin imzanı taşıyan bir kâğıda kendi
isteğini yazdırırsa okul yine uygular. Tarayıcı da aynen böyle: login olduğun siteye giden **her
isteğe session cookie'yi otomatik ekler** — saldırgan bu otomatizmi sömürür.

## CSRF'in üç şartı (bu lab'da üçü de vardı)

1. **Değer taşıyan işlem:** e-posta değiştirme (`change-email`).
2. **Sadece cookie ile kimlik:** istekte başka gizli doğrulama (CSRF token) yok.
3. **Tahmin edilebilir parametreler:** tek parametre `email`, saldırgan isteği önceden kurabiliyor.

## Hedef istek (Burp'ten)

```
POST /my-account/change-email HTTP/2
Host: <lab>.web-security-academy.net
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=wiener@normal-user.net
```

> Savunmasızlık kararı: gövdede de header'da da **CSRF token YOK** + kimlik **sadece cookie**.
> Token olsaydı saldırgan onu bilemez, saldırı çökerdi.

## Exploit (elle kurulan saldırgan sayfası)

```html
<html>
  <body>
    <form action="https://<lab>.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked@evil.com">
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

Her satırın görevi:
- **`action`** = isteğin gideceği **tam URL + path**. Absolute olmalı; çünkü form saldırganın
  sunucusunda duruyor ama istek **kurbanın sitesine** gitmeli.
- **`method="POST"`** = hedef istekle birebir aynı method.
- **`<input hidden name="email">`** = asıl **payload (veri)**. `type=hidden` sadece görünmez
  yapar (kurban fark etmesin); işi taşıdığı **name/value** çiftidir. Değer, mevcut e-postadan
  **farklı** olmalı ki değişiklik kanıtlansın.
- **`document.forms[0].submit()`** = otomatik tetik. Kurban sayfayı açar açmaz form kendiliğinden
  gönderilir; kurbanın tıklamasına gerek yok.

> `Content-Type` hedefte `x-www-form-urlencoded`'du — düz bir HTML `<form>` zaten tam bu formatı
> üretir. Bu yüzden `fetch`/JS gerekmedi; savunmasız CSRF'in bu kadar basit olmasının sebebi bu.

## Arka planda ne oldu? (CSRF'in kalbi)

1. Kurban saldırgan sayfasını açtı.
2. Script formu otomatik submit etti → tarayıcı `change-email` isteğini **web-security-academy.net**'e gönderdi.
3. **Kritik:** İstek o domain'e gittiği için tarayıcı, kurbanın o siteye ait **session cookie'sini
   otomatik ekledi** (kurban login olduğundan cookie hazırdı).
4. Sunucu geçerli cookie'yi gördü, kontrol edecek **token olmadığı için** "kullanıcı istedi" sandı
   → e-postayı değiştirdi → **lab çözüldü.**

**Özet:** İsteği saldırgan kurdu, ama "imzayı" (cookie) **kurbanın tarayıcısı otomatik attı.**

## Remediation (defender tarafı)

1. **CSRF token (en etkili):** Her state-changing formda, sunucunun ürettiği **tahmin edilemez,
   oturuma bağlı** gizli bir token. Sunucu her istekte doğrular. Saldırgan token'ı bilemez →
   saldırı çöker. (Modern framework'lerde çoğu zaman hazır: Django `{% csrf_token %}`, ASP.NET
   `[ValidateAntiForgeryToken]`, Spring Security CSRF.)
2. **SameSite cookie:** Session cookie'ye `SameSite=Lax` (veya `Strict`). Böylece tarayıcı,
   **başka siteden** tetiklenen isteklere cookie'yi otomatik eklemez — CSRF'in otomatizmini kırar.
3. **Origin/Referer kontrolü:** Sunucu, isteğin kendi origin'inden geldiğini doğrulasın (ikincil
   savunma; tek başına yeterli değil).
4. **Kritik işlemlerde re-authentication:** Şifre/e-posta değişiminde mevcut şifreyi tekrar iste.

## Zincir bağı (önceki konularla)

- **Injection / Info Disclosure:** sen veri gönderiyor ya da okuyordun.
- **CSRF:** veri yok — **kurbanın oturumunu** kullanıyorsun. Yeni bir eksen: güven/kimlik istismarı.
- **Authentication** (session cookie) + **Access Control** (kurbanın yetkisiyle işlem) kesişimi.
- Client-side bir saldırı; kurbanın tarayıcısı saldırı aracı olur (XSS ile akraba düşünce).

## Köşe taşı

> Bir state-changing istek **sadece cookie** ile doğrulanıyor ve **tahmin edilemez bir token
> taşımıyorsa**, o işlem CSRF'e açıktır. Savunmanın özü: "bu isteği gerçekten kullanıcı mı
> başlattı?" sorusunu cookie'den **bağımsız** bir kanıtla (token) yanıtlamak.
