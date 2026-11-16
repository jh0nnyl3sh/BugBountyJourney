# Spiral Tekrar Günlüğü

> Amaç: Eski konuları **elle** (notu ezberleyerek değil, lab'ı sıfırdan yeniden çözerek) tazelemek.
> Her tekrar sonrası kısa not: **ne hatırladım, ne unutmuştum, hangi incelik eksikti.**
> Unutma eğrisine karşı en iyi silah tekrar — ve tekrar verbal değil, hands-on olmalı.

## Tekrarlar

| # | Tarih | Konu | Yeniden çözülen lab | Durum | Not |
|---|-------|------|---------------------|-------|-----|
| 1 | 14.11.2026 | SSRF | Basic SSRF against the local server | 🔄 tazelendi | Dün SSRF'i unutmuştum (XSS ile karıştırdım). Re-solve ile geri geldi; payload nokta atışı (`stockApi=http://localhost/admin/delete?username=carlos`). Düzeltilen incelik: SSRF isteğini **ben değil SUNUCU** atar — ben sadece sunucunun gideceği URL'i ele geçiririm. "Postacı benim yerime içeride gezer." |

## Takip: hangi konu ne zaman tazelenmeli?

- **SSRF** → 14.11 tazelendi ✅ (dün tamamen unutmuştum — tekrar aralığını sık tut)
- **SQL Injection** → en çok lab (11) ama bir süredir el değmedi; yakında bir blind SQLi re-solve
- **XSS** → 16.11 tazelendi ✅ (payload vardı, impact/"neden tehlikeli" boşluğu kapatıldı); pusulada hâlâ puanlanmadı
- **Access Control / IDOR** → orta tazelik
| 2 | 15.11.2026 | Access Control (A01) | Unprotected admin functionality (AC #5) | 🔄 tazelendi | OWASP A01 gününde re-solve. GELİŞİM KANITI: 12.10'da robots.txt'i bilmiyordu, sormuştu; bugün /admin çalışmayınca **kendi başına** robots.txt'e gitti → /administrator-panel → carlos. Info Disclosure + Broken Access Control zincirini kendi kurdu. |
| 3 | 16.11.2026 | XSS (Cross-site scripting) | Reflected XSS into HTML context, nothing encoded (XSS #1) | 🔄 tazelendi | Payload hemen geldi (`<script>alert(1)</script>`) ama "neden tehlikeli?" boşluğu vardı → öğretildi. Kendi cümlesi: kod kurbanın tarayıcısında çalışır → cookie/oturum/sayfa verisi/yazılanlar çalınır. Precision: JS kayıtlı şifre deposunu okuyamaz ama cookie + yazılan veriyi alır. XSS = "CSRF'in güçlü versiyonu; token'ı bile okur". |
