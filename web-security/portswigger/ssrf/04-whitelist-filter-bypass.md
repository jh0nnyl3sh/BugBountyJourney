# SSRF Lab #4: whitelist (beyaz liste) atlatma — 05.11.2026

> İlk Expert lab. Whitelist "sadece şu host'a izin var" diyor; biz URL'nin validator ile istek-client'ı tarafından FARKLI ayrıştırılmasını (parser disagreement) kullanarak aşıyoruz. @ (userinfo) + # (fragment) + çift URL-encode üçlüsü.

**Lab:** PortSwigger → SSRF → *SSRF with whitelist-based input filter* (Expert, Java-URL quirk)
**Görev:** Whitelist'i aşıp localhost admin'e ulaş, carlos'u sil.

## Durum
- `localhost/admin`, `127.0.0.1/admin` → **400** (whitelist: "beklenen host değilsin").
- Basit `@` (`stock...@localhost`) → yine engellendi (akıllı parser gerçek host'u görüyor).
- Tek `%23` (`stock...%23@127.1`) → **500**: filtre geçti (host=stock, `#` fragment) ama istek de **stock**'a gitti (yanlış hedef). "Geçmek ≠ ulaşmak".

## Çözüm (doğrulanmış payload)
```
stockApi=http://%25%36%63%25%36%66%25%36%33%25%36%31%25%36%63%25%36%38%25%36%66%25%37%33%25%37%34%25%32%33@stock.weliketoshop.net/admin/delete?username=carlos
```
O uzun blok = **`localhost#`'in çift URL-encode'u** (`l`→`%6c`→`%25%36%63` ... `#`→`%23`→`%2523`). carlos silme yolu zaten içinde → tek atış. **Solved.** ✅

## Neden çalışıyor — parser disagreement (asıl kavram)
Aynı URL metnini iki bileşen **farklı** ayrıştırıyor:
- **Güvenlik filtresi:** `@`'in solundaki blok "kullanıcı adı" (userinfo), host = `@`'in sağı = **`stock.weliketoshop.net`** → whitelist GEÇER.
- **İsteği atan client:** blok decode olunca `localhost#` olur → `#` fragment başlatır, gerçek host **`localhost`** → istek localhost'a gider.
Üç araç bu ayrışmayı üretir:
- **`@` (userinfo):** `A@B` → host B; whitelist host'u `@`'in sağına koy.
- **`#` (fragment):** decode sonrası host sınırını erken kapatıp host'u localhost yapar.
- **Çift encode:** blok filtreye opak görünür (bir decode, host=stock); istek aşamasında bir kez daha decode olup `localhost#` olur. İki katmanın decode farkı = açık.

## Dürüst sınır
Hangi katmanın tam kaç kez decode ettiği ve `/admin/delete` yolunun fragment'e rağmen nasıl gittiği, PortSwigger'ın Java URL/URI kütüphanesinin **bilinen tuhaflığına** bağlı; bayt-bayt %100 trace iddia etmiyoruz. Kalıcı ders **kavram**: whitelist bile, **validator ≠ requester parser** ise kırılır.

## 🛡️ Nasıl kapatılır (remediation)
- **Tek ve tutarlı parser:** doğrulama ile isteği atan kod **aynı** URL ayrıştırıcıyı kullanmalı; biri userinfo/fragment'i şöyle, öbürü böyle yorumlamamalı.
- **Host'u çöz, IP'yi doğrula:** URL'yi parse edip host'u IP'ye çöz, çözülen IP iç/loopback ise reddet. (Aşmaya karşı asıl sağlam nokta burası — metin değil, çözülen IP.)
- **userinfo/fragment'i reddet:** `@` ve `#` içeren, ya da birden çok decode gerektiren URL'leri kabul etme.
- **En iyisi: kullanıcıya tam URL verdirme** — bir ID/seçenek al, iç URL'yi sunucu sabit şablonla kursun.

## Zincir bağı
- Blacklist (#3) → whitelist (#4): "aynı hedef, farklı kılık"ın en ileri hali. Buradaki "iki parser'ı anlaşmazlığa düşür" fikri, ileride WAF/parser atlatmanın temeli.
