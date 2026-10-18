# 🛡️ Bug Bounty Journey

> Sıfırdan etik hacker'lığa / bug bounty avcılığına uzanan kişisel öğrenme yolculuğunun günlüğü.

**Başlangıç:** 06.10.2026 · **Bir yıllık kontrol noktası:** 06.10.2027

---

## 📊 Scoreboard

### ✅ Toplam Çözülen Lab: **12**

| Konu | Çözülen | Durum |
|------|:-------:|-------|
| Access Control / IDOR | 7 | 🔄 devam ediyor |
| Cross-Site Scripting (XSS) | 5 | 🔄 devam ediyor |
| SQL Injection | 0 | ⏳ sırada |
| SSRF | 0 | ⏳ sırada |
| Authentication | 0 | ⏳ sırada |
| CSRF | 0 | ⏳ sırada |

### 🧭 İlerleme

- [x] **Temeller** — Web/HTTP/DNS, oturum (session), Burp Suite
- [x] **Etik çizgi** — scope & yetki refleksi
- [x] **Access Control / IDOR** — veri + işlev/rol ekseni (7 lab)
- [ ] **XSS** — context'ler: HTML ✅, attribute ✅, JS string ✅, DOM ✅ · (stored-ileri, filtre atlatma sürüyor)
- [ ] **SQL Injection**
- [ ] **SSRF**
- [ ] **Authentication zafiyetleri**
- [ ] **CSRF**
- [ ] **Python ile otomasyon** (payload/recon)

---

## 🎯 Hedef

Web uygulama güvenliğinde sağlam bir temel kurup, izinli ve etik sınırlar içinde (bug bounty programları + yasal lab'lar) zafiyet avcılığı yapabilecek seviyeye gelmek.

**Felsefe:** Hedef bug bounty (motivasyon), günlük iş ise basamakları sırayla ve sindirerek geçmek. Para, bilginin doğal sonucudur. 100 kiloyu bir kere değil, 1 kiloyu 100 kere.

## 🗺️ Yol Haritası

1. **Temeller** — İnternet/web nasıl çalışıyor (HTTP, DNS, oturum)
2. **Araçlar** — Burp Suite başta olmak üzere
3. **Zafiyet sınıfları** — OWASP Top 10 ve ötesi (IDOR, XSS, SQLi, SSRF...)
4. **Pratik** — PortSwigger Academy, TryHackMe, HackTheBox (yasal lab'lar)
5. **Gerçek sahne** — HackerOne, Bugcrowd, Intigriti

**Bilgi basamakları:** Ağ + Linux temelleri → Web uygulama güvenliği (OWASP + Burp) → Bir alanda uzmanlaşma
**Sertifika hedefi (şimdilik):** BSCP (Burp Suite Certified Practitioner)

---

## 📁 Repo Yapısı

```
.
├── temeller/                     platform-bağımsız bilgi (konseptler)
└── web-security/
    └── portswigger/              platform
        ├── access-control/       konu
        └── cross-site-scripting/ konu
```

İleride TryHackMe, HackTheBox gibi platformlar `web-security/` altına; web dışı alanlar ise `web-security`'nin kardeşi olarak eklenir.

## 📚 Temeller (platform-bağımsız)

Her yeni konuya geçmeden önce ilgili dosyayı bir oku, tekrar et, sonra devam et.

| # | Konu | Dosya |
|---|------|-------|
| 1 | Web Nasıl Çalışır? (İstek/Yanıt Döngüsü) | [temeller/01-web-nasil-calisir.md](./temeller/01-web-nasil-calisir.md) |
| 2 | HTTP İsteğinin Anatomisi | [temeller/02-http-istegi-anatomisi.md](./temeller/02-http-istegi-anatomisi.md) |
| 3 | Oturum (Session) Mantığı | [temeller/03-oturum-session-mantigi.md](./temeller/03-oturum-session-mantigi.md) |
| 4 | IDOR (kavram) | [temeller/04-idor.md](./temeller/04-idor.md) |
| 5 | Burp Suite (İsteği Havada Yakalamak) | [temeller/05-burp-suite.md](./temeller/05-burp-suite.md) |
| ⚖️ | Etik Çizgi (her zaman cepte) | [temeller/etik-cizgi.md](./temeller/etik-cizgi.md) |

## 🧪 Lab Çözümleri — web-security / portswigger

### Access Control

| # | Lab | Dosya |
|---|-----|-------|
| 1 | Insecure direct object references | [01-insecure-direct-object-references.md](./web-security/portswigger/access-control/01-insecure-direct-object-references.md) |
| 2 | User ID controlled by request parameter | [02-user-id-request-parameter.md](./web-security/portswigger/access-control/02-user-id-request-parameter.md) |
| 3 | ...with unpredictable user IDs | [03-unpredictable-ids.md](./web-security/portswigger/access-control/03-unpredictable-ids.md) |
| 4 | ...with data leakage in redirect | [04-data-leakage-redirect.md](./web-security/portswigger/access-control/04-data-leakage-redirect.md) |
| 5 | Unprotected admin functionality | [05-unprotected-admin.md](./web-security/portswigger/access-control/05-unprotected-admin.md) |
| 6 | Unprotected admin functionality with unpredictable URL | [06-admin-url-js-leak.md](./web-security/portswigger/access-control/06-admin-url-js-leak.md) |
| 7 | User role controlled by request parameter | [07-role-cookie.md](./web-security/portswigger/access-control/07-role-cookie.md) |

### Cross-Site Scripting (XSS)

| # | Lab | Dosya |
|---|-----|-------|
| 1 | Reflected XSS — HTML context, nothing encoded | [01-reflected-html-context.md](./web-security/portswigger/cross-site-scripting/01-reflected-html-context.md) |
| 2 | Stored XSS — HTML context, nothing encoded | [02-stored-html-context.md](./web-security/portswigger/cross-site-scripting/02-stored-html-context.md) |
| 3 | Reflected XSS — attribute, angle brackets encoded | [03-attribute-angle-brackets-encoded.md](./web-security/portswigger/cross-site-scripting/03-attribute-angle-brackets-encoded.md) |
| 4 | Reflected XSS — JavaScript string breakout | [04-javascript-string-breakout.md](./web-security/portswigger/cross-site-scripting/04-javascript-string-breakout.md) |
| 5 | DOM XSS — document.write sink (source: location.search) | [05-dom-xss-document-write.md](./web-security/portswigger/cross-site-scripting/05-dom-xss-document-write.md) |

## 🏆 Çözülen Lab'lar (kronolojik)

| No | Tarih | Platform | Konu | Lab | Seviye |
|:--:|-------|----------|------|-----|--------|
| 1 | 09.10.2026 | PortSwigger | Access Control | Insecure direct object references | Apprentice |
| 2 | 10.10.2026 | PortSwigger | Access Control | User ID controlled by request parameter | Apprentice |
| 3 | 11.10.2026 | PortSwigger | Access Control | ...with unpredictable user IDs | Apprentice |
| 4 | 11.10.2026 | PortSwigger | Access Control | ...with data leakage in redirect | Apprentice |
| 5 | 12.10.2026 | PortSwigger | Access Control | Unprotected admin functionality | Apprentice |
| 6 | 14.10.2026 | PortSwigger | Access Control | Unprotected admin functionality with unpredictable URL | Apprentice |
| 7 | 14.10.2026 | PortSwigger | Access Control | User role controlled by request parameter | Apprentice |
| 8 | 15.10.2026 | PortSwigger | XSS | Reflected XSS into HTML context with nothing encoded | Apprentice |
| 9 | 15.10.2026 | PortSwigger | XSS | Stored XSS into HTML context with nothing encoded | Apprentice |
| 10 | 16.10.2026 | PortSwigger | XSS | Reflected XSS into attribute with angle brackets HTML-encoded | Apprentice |
| 11 | 17.10.2026 | PortSwigger | XSS | Reflected XSS into a JavaScript string with angle brackets HTML encoded | Apprentice |
| 12 | 18.10.2026 | PortSwigger | XSS | DOM XSS in document.write sink using source location.search | Apprentice |

## 🔜 Sıradaki Adım

- XSS: DOM-based ileri sink'ler (innerHTML, eval, jQuery), stored/filtre atlatma
- İleride: XSS payload'larını otomatikleştirme (Python), context'e göre payload seçimi

---

*Bu repo bir öğrenme günlüğüdür; içinde gerçek hedeflere dair somut zafiyet bilgisi barındırmaz, yalnızca genel öğrenme notları içerir.*
