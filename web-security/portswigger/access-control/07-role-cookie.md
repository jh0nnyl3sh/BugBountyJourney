# 12. Access Control Lab #7: Rol çerezde kontrol ediliyor — 14.10.2026

> Access control'ün "rol" ekseni. Dünkü kuralın canlı hali: değiştirebildiğin şey (çerez), güvenmemen gereken şeydir.

**Lab:** PortSwigger → Access control → *User role controlled by request parameter* (Apprentice)
**Görev:** `wiener:peter` ile gir, admin paneline eriş, `carlos`'u sil.

## Çözüm (körleme, elle)
1. `wiener:peter` ile giriş. `/admin` denendi → "Admin interface only available to administrators" + **401 Unauthorized**.
2. Burp HTTP history'de `/admin` isteği incelendi → request içinde çerez: **`Admin=false`**.
3. **Kanıt:** history'de `Admin=false` → `true` yapılıp gönderildi → yanıt **401 → 200 OK** oldu. Ama tarayıcıya yansımadı (çünkü bu tek seferlik kopya Burp içinde kaldı, canlı oturuma dokunmadı).
4. **Sömürü:** Intercept açıldı, tarayıcıdan giden canlı istek yakalandı, `Admin=false` → `true` yapıldı → tarayıcı gerçek admin panelini aldı → carlos silindi → **Solved.** ✅

## Kanıt vs sömürü (önemli ayrım)
- **HTTP history / Repeater'da düzenlemek** = tek seferlik kopya gönderir; yanıt Burp'te kalır. Zafiyeti **kanıtlar**, canlı oturumu değiştirmez.
- **Intercept ile canlı isteği değiştirmek** = tarayıcının gerçek isteğini değiştirir; sonuç tarayıcıya yansır. **Sömürür.**
- Pratikte: önce history/Repeater ile kanıtla, gerekirse canlı sömür.

## Zincir (bağlar)
- Dün (#6): "client-side gizli değildir" + "değiştirebildiğin şey güvenilmezdir."
- Bugün: çerez senin kontrolünde → site yetki kararını oraya bırakmış.
- Status code okuma (401/200), ilk IDOR günündeki "yanıtın nasıl olduğu da ipucudur" dersinin devamı.

## Kök sebep
- Çerez artık sadece **kimlik** (session ID) değil, **rol/yetki** (`Admin=...`) de taşıyor; bu client-controlled.
- Kural: yetki bilgisi asla client'ta tutulmaz. Sunucu "bu kullanıcı admin mi?" sorusunu kendi **veritabanından** cevaplamalı, çerezdeki bir yazıya bakarak değil.
