# Access Control Lab #8: User ID request parameter + password disclosure — 07.11.2026

> Gerçek-hedef odaklı dönüş: IDOR + password disclosure zinciri. VDP'lerin giriş kapısı sınıfı — tekrar + gerçek dünya pratiği.

**Lab:** PortSwigger → Access control → *User ID controlled by request parameter with password disclosure* (Apprentice)
**Görev:** administrator'ın şifresini ele geçir, giriş yap, carlos'u sil.

## İki zafiyet zinciri
1. **IDOR:** `wiener:peter` ile girip hesap sayfasında `id=administrator` yapınca → admin'in hesap sayfası geliyor. Sunucu "bu sayfa senin mi?" **yetki kontrolünü yapmıyor.**
2. **Password disclosure:** O sayfada şifre alanı ekranda `••••` ile gizli görünüyor — ama **HTML kaynağında** input'un `value="..."` özniteliğinde **düz metin.** (Inspect / View Source ile görünür.)

## Çözüm
1. `/my-account?id=administrator` aç.
2. Şifre input'una sağ tık → **Inspect** → `value` içinde admin'in düz-metin şifresi.
3. Çıkış → `administrator` + o şifreyle giriş → admin paneli → **carlos silindi** → **Solved.** ✅
(Not: sayfadaki şifre *değiştirme* formu giriş yapmış kullanıcı için çalışır — hedef o değildi; şifreyi **okumak** yeterliydi.)

## Ders
- **Client-side gizli değildir:** ekranda `••••` sadece kozmetik; client'a giden HTML'de her şey görünür. ("Göster"ilmeyen ≠ "gönderil"meyen.)
- İki küçük hata tek başına sınırlıyken **zincirlenince** tam hesap devralma oldu — gerçek bug bounty'de etki çoğu zaman zincirden gelir.

## 🛡️ Nasıl kapatılır (remediation)
- **Nesne-bazlı yetki (server-side):** her istekte "bu kullanıcı bu kaynağa erişebilir mi?" kontrol et; `id` parametresine körlemesine güvenme.
- **Hassas veriyi client'a gönderme:** şifre (ya da token, PII) yanıt HTML'inde/alan value'sunda **hiç bulunmamalı.** Maskeleme çözüm değil.
- **Şifreleri geri çevrilemez hash'le** (bcrypt/Argon2) → sızsa bile plaintext yok.

## Zincir bağı
- Erken access-control lab'larının (id/parametre ile IDOR) devamı; buraya "password disclosure" katmanı eklendi.
- Gerçek-hedef köprüsüne bağ: IDOR + info/credential disclosure, acemi VDP avcısının en sık bulduğu ikili.
