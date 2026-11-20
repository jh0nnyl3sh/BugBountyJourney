# Business Logic #1: Excessive trust in client-side controls

- **Seviye:** Apprentice
- **Sınıf:** Business Logic (yeni Tier-1 sınıf)
- **Yöntem:** Körleme (parametre tampering)
- **Araç:** Burp Repeater + tarayıcı checkout

---

## Business Logic nedir?

Uygulamanın **mantığındaki / varsayımlarındaki** hatadır — teknik bir injection değil. Uygulama,
garanti olmayan bir şeyi garanti sanır. En sık hali: **istemciden gelen bir değere körü körüne güvenmek.**

### Çocuğa anlatır gibi
Bir otomat, parayı kontrol etmek yerine "kaç lira attın?" diye **sana sorar** ve ne dersen inanır.
Sen "100 lira" dersin, o inanır. İşte bu lab: sunucu ürünün **fiyatını senin gönderdiğin değere** güveniyor.

## Çözüm

- Mağazada pahalı bir ceket var; store credit bakiyem yetmiyor.
- Sepete ekleme isteğini (POST `/cart`) Burp'te yakaladım → içinde **`price`** parametresi var.
- Repeater'da `price`'ı **bakiyeme sığacak** düşük bir değere çektim → Send.
- İstek benim session cookie'mle gittiği için ürün **gerçek sepetime o düşük fiyatla** eklendi.
- Tarayıcıda **checkout** → bakiye yettiği için sipariş geçti → **ceketi girdiğim fiyattan aldım** → çözüldü.

## Kök neden (kusur)

> Sunucu, ürünün fiyatını **istemciden gelen `price`'a** güvenerek belirledi; kendi tarafında
> **doğrulama yoktu.** Oysa `price` saldırganın kontrolünde.

## Büyük resim bağı (çekirdek ilke)

Tüm zafiyetlerin kökü aynı: **uygulama, doğrulanmamış girdiye güvenir.**
- Injection'da o girdi **koda** dönüşür (SQLi→query, XSS→HTML, Command Inj→shell).
- Business logic'te o girdi bir **değere/varsayıma** dönüşür (price).
- Sink farklı, **kök aynı: istemciden gelene güvenme.** Business logic, bu ilkenin değerlere uygulanmış hali.

## Neden "injection değil"?

Bir şey enjekte etmiyorsun — uygulamanın **mantık boşluğunu** buluyorsun. Payload ezberi yok;
"uygulama neyi garanti sanıyor ama değil?" diye **düşünmek** var. Bu yüzden otomatik tarayıcı
business logic bulamaz — **insan muhakemesi** bulur.

## Remediation (defender)

1. **Fiyatı sunucu belirlesin:** İstemciden sadece **`productId`** (ve miktar) alınmalı; fiyat
   **sunucunun kendi veritabanından** bakılmalı. İstemciden gelen `price` asla kullanılmamalı.
2. **Genel ilke:** Güvenlik/iş açısından kritik hiçbir değer istemciye bırakılmaz; her kritik karar
   sunucuda, güvenilir kaynaktan doğrulanır.
3. **Sunucu tarafı iş kuralı doğrulaması:** toplam, indirim, stok, yetki — hepsi sunucuda teyit edilir.

## Köşe taşı

> İstemciden gelen her değer saldırganın kontrolündedir. Fiyat, rol, miktar, "isAdmin" gibi kritik
> hiçbir şey istemciye emanet edilmez — sunucu her kritik değeri **kendi güvenilir kaynağından** doğrular.
