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
| — | ⚖️ Etik Çizgi (her zaman cepte) | [konular/etik-cizgi.md](./konular/etik-cizgi.md) |

## 🔜 Sıradaki Adım

- Burp Suite kurulumu ve ilk kullanım (Kali VM üzerinde)
- Ardından PortSwigger Web Security Academy'de ilk IDOR lab'ını elle sömürmek

---

*Bu repo bir öğrenme günlüğüdür; içinde gerçek hedeflere dair somut zafiyet bilgisi barındırmaz, yalnızca genel öğrenme notları içerir.*
