# 🛡️ Bug Bounty Journey

> Sıfırdan etik hacker'lığa / bug bounty avcılığına uzanan kişisel öğrenme yolculuğunun günlüğü.

**Başlangıç:** 06.10.2026
**Bir yıllık kontrol noktası:** 06.10.2027

---

## 🎯 Hedef

Web uygulama güvenliğinde sağlam bir temel kurup, izinli ve etik sınırlar içinde (bug bounty programları + yasal lab'lar) zafiyet avcılığı yapabilecek seviyeye gelmek.

**Felsefe:** Hedef bug bounty (motivasyon), günlük iş ise basamakları sırayla ve sindirerek geçmek. Para, bilginin doğal sonucudur.

## 🗺️ Yol Haritası

1. **Temeller** — İnternet/web nasıl çalışıyor (HTTP, DNS, oturum)
2. **Araçlar** — Burp Suite başta olmak üzere
3. **Zafiyet sınıfları** — OWASP Top 10 ve ötesi (IDOR, XSS, SQLi, SSRF...)
4. **Pratik** — PortSwigger Academy, TryHackMe, HackTheBox (yasal lab'lar)
5. **Gerçek sahne** — HackerOne, Bugcrowd, Intigriti

**Bilgi basamakları:** Ağ + Linux temelleri → Web uygulama güvenliği (OWASP + Burp) → Bir alanda uzmanlaşma
**Sertifika hedefi (şimdilik):** BSCP (Burp Suite Certified Practitioner)

## 📚 Konular

Her yeni konuya geçmeden önce ilgili dosyayı bir oku, tekrar et, sonra devam et.

| # | Konu | Dosya |
|---|------|-------|
| 1 | Web Nasıl Çalışır? (İstek/Yanıt Döngüsü) | [konular/01-web-nasil-calisir.md](./konular/01-web-nasil-calisir.md) |
| 2 | HTTP İsteğinin Anatomisi | [konular/02-http-istegi-anatomisi.md](./konular/02-http-istegi-anatomisi.md) |
| 3 | Oturum (Session) Mantığı | [konular/03-oturum-session-mantigi.md](./konular/03-oturum-session-mantigi.md) |
| 4 | IDOR | [konular/04-idor.md](./konular/04-idor.md) |
| 5 | Burp Suite (İsteği Havada Yakalamak) | [konular/05-burp-suite.md](./konular/05-burp-suite.md) |
| 6 | 🎉 İlk Çözülen Lab: IDOR (dosya tabanlı) | [konular/06-ilk-idor-lab.md](./konular/06-ilk-idor-lab.md) |
| 7 | IDOR Lab #2: User ID / URL parametresi (+ Repeater) | [konular/07-idor-lab-user-id-parametresi.md](./konular/07-idor-lab-user-id-parametresi.md) |
| — | ⚖️ Etik Çizgi (her zaman cepte) | [konular/etik-cizgi.md](./konular/etik-cizgi.md) |

## 🏆 Çözülen Lab'lar

| Tarih | Platform | Lab | Seviye |
|-------|----------|-----|--------|
| 09.10.2026 | PortSwigger | Insecure direct object references | Apprentice |
| 10.10.2026 | PortSwigger | User ID controlled by request parameter | Apprentice |

## 🔜 Sıradaki Adım

- IDOR: *User ID controlled by request parameter, with unpredictable user IDs* (karmaşık/sızan ID)
- Access control / Apprentice lab'larını sırayla tamamlamak

---

*Bu repo bir öğrenme günlüğüdür; içinde gerçek hedeflere dair somut zafiyet bilgisi barındırmaz, yalnızca genel öğrenme notları içerir.*
