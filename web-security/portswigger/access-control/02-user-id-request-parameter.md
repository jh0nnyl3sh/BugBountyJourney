# 7. IDOR Lab #2: User ID controlled by request parameter — 10.10.2026

> İkinci çözülen lab. Dünkü dosya tabanlı IDOR'un kardeşi: bu kez "nesne" bir kullanıcı hesabı, başvuru ise URL parametresi.

**Lab:** PortSwigger → Access control → *User ID controlled by request parameter* (Apprentice)
**Görev:** `carlos` kullanıcısının **API key**'ini bul ve çözüm olarak gönder.
**Verilen hesap:** `wiener:peter`

## Sömürü adımları
1. `wiener:peter` ile giriş yapıldı → My account sayfasında kendi API key'im göründü.
2. URL'de parametre: `/my-account?id=wiener`.
3. **Avcı refleksi:** `id` doğrudan kullanıcıyı belirliyor → değiştirirsem başkasının hesabına düşer miyim?
4. `id=wiener` → `id=carlos` yapıldı.
5. Sayfa carlos'un hesabını açtı; My account kısmında **carlos'un API key'i** göründü.
6. Key, "Submit solution" ile gönderildi → **Solved.** ✅

## Aynısı Burp Repeater ile (tekrar, kas için)
- Proxy → HTTP history → `/my-account?id=wiener` isteği → sağ tık → **Send to Repeater**.
- Repeater'da ilk satırda `wiener` → `carlos` yapıldı, **Send**.
- Sağdaki yanıtın HTML gövdesinde carlos'un API key'i görüldü.
- **Neden Repeater?** URL değiştirmek sadece basit GET'lerde çalışır. POST gövdesi, başlıklar, çerezler adres çubuğundan değiştirilemez; Repeater her tür isteği alıp istenen yeri değiştirip tekrar tekrar göndermeyi sağlar.

## Teorik karşılığı
- **Nesne:** kullanıcı hesabı. **Doğrudan başvuru:** `?id=<kullanıcı>` parametresi.
- **Kök sebep:** sunucu "bu hesap sana mı ait?" diye **authorization** kontrolü yapmıyor.
- **Tür:** yatay yetki yükseltme (horizontal privilege escalation).

## Not — gerçeklik payı
Bu lab hızlı çözüldü çünkü: (1) Apprentice seviye, tek kavram; (2) hesap + görev + tek açık endpoint hazır verildi; (3) işin %90'ı olan "açığı bulma/keşif" kısmı atlanmıştı. Gerçek bug bounty'de zor olan, zafiyeti sömürmek değil, hangi endpoint'te olduğunu bulmaktır.
