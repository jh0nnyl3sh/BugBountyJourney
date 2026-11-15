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
- **XSS** → 7 lab, uzun süredir dokunulmadı; ilgi pusulasında bile puanlanmamış — tazeleme sırada
- **Access Control / IDOR** → orta tazelik
| 2 | 15.11.2026 | Access Control (A01) | Unprotected admin functionality (AC #5) | 🔄 tazelendi | OWASP A01 gününde re-solve. GELİŞİM KANITI: 12.10'da robots.txt'i bilmiyordu, sormuştu; bugün /admin çalışmayınca **kendi başına** robots.txt'e gitti → /administrator-panel → carlos. Info Disclosure + Broken Access Control zincirini kendi kurdu. |
