# XSS Lab #7: Stored — anchor href, çift tırnak HTML-encoded (javascript: şeması) — 30.10.2026

> Sarmal/tekrar lab'ı (erken konuya elle dönüş). XSS'in "context payload'ı belirler" dersini tazeliyor ama yeni bir context'le: breakout kapalıyken context'in kendi mekanizmasını (href'in `javascript:` şeması) kötüye kullanmak.

**Lab:** PortSwigger → XSS → Contexts → *Stored XSS into anchor href attribute with double quotes HTML-encoded* (Apprentice)
**Görev:** `alert` çağıran bir XSS gerçekleştir.

## Context ve neden breakout yok
- Yorum formundaki **Website** alanına yazdığın değer, yorumunla görünen isim linkinin href'i oluyor: `<a href="GİRDİ">isim</a>`.
- Breakout denendi (`"` ile href'ten, `>` ile tag'den çıkmak) → **çift tırnak ve açılı parantezler HTML-encode** ediliyor (`&quot;`, `&lt;`) → yeni tag sokmak **kapalı.**

## Çözüm
Website alanına:
```
javascript:alert(1)
```
Sonuç DOM'da:
```html
<a href="javascript:alert(1)">isim</a>
```
Link **tıklanınca** tarayıcı href'teki JavaScript'i çalıştırır → alert → **Solved.** ✅
(Enjeksiyon normal form üzerinden gittiği için tamamen tarayıcıdan yapıldı; Burp gerekmedi.)

## Mantık (tazelenen ders)
- Breakout (yeni tag sokma) kapalıysa, **context'in kendi sunduğu mekanizmayı** kullan. Burada o mekanizma href'in `javascript:` şeması: bir link tıklanınca `javascript:` ile başlayan href'i çalıştırır.
- "Context payload'ı belirler" dersinin ileri hali: çözüm her zaman tag açmak değildir; bazen zaten orada olan bir özelliği kötüye kullanmaktır. (Filtre/WAF atlatmanın temel mantığı da budur.)
- Ne zaman Burp, ne zaman tarayıcı: girdi normal bir formdan gidiyorsa tarayıcı yeter; çerez/header/otomasyon gerekiyorsa Burp.

## 🛡️ Nasıl kapatılır (remediation)
- **URL şema allowlist:** href'e yalnızca güvenli şemalara (`http`, `https`, `mailto`) izin ver; `javascript:`, `data:`, `vbscript:` gibi şemaları reddet. (Encode etmek tek başına yetmedi; çünkü sorun tırnak değil, şemaydı.)
- **Context'e uygun çıktı + framework sanitizasyonu:** modern framework'ler (ör. Angular, React) URL binding'de bu şemaları otomatik temizler; ham string href kurarken kendin doğrula.
- Genel kural: kullanıcı girdisi bir URL olacaksa, **parse edip şemasını kontrol et** — "veri, kod değil".

## Zincir bağı
- Önceki XSS context lab'larında (#3 attribute, #4 JS string) breakout ile çözüyorduk. Burada breakout kapalı → context'in kendi silahı (`javascript:`).
- Bu, ileride "most tags blocked / filtre atlatma" lab'larının zihniyetinin ilk tadı.
