# 11. Access Control Lab #6: Admin URL leaked in JavaScript — 14.10.2026

> Bir öncekinin (robots.txt) kardeşi: gizli admin adresi bu kez sayfanın kendi JavaScript kodunda sızıyor. Ders: client-side'a giden hiçbir şey gizli değildir.

**Lab:** PortSwigger → Access control → *Unprotected admin functionality with unpredictable URL* (Apprentice)
**Görev:** Gizli admin panelini bul, erişip `carlos`'u sil.

## Çözüm (körleme, elle)
1. Site gezildi; Home ve My account var. My account → login ekranı.
2. Sayfanın HTML/JavaScript kaynağı incelendi (View Page Source).
3. JS içinde bir `if (isAdmin)` bloğu vardı; admin panelinin gerçek yolu orada gömülü: `/admin-ndjeqa` (rastgele ekli, "tahmin edilemez" URL).
4. O yola gidildi → panel açıldı → carlos silindi → **Solved.** ✅

## Zincir (önceki laba bağ)
- Lab #5: gizli admin adresi `robots.txt`'de (paspasın altındaki anahtar).
- Lab #6: aynı gizli adres bu kez **JavaScript** içinde sızıyor.
- Ortak mantık: "gizli işlev, adresini bir yerde ifşa eder." Sızıntının yeri değişir, mantık aynı kalır.

## Altın ders — client-side gizli değildir
- Tarayıcıya giden her şey (HTML, CSS, JS) kullanıcının makinesinde çalışır → kullanıcı hepsini görebilir.
- Admin URL'i, API key, şifre ya da `isAdmin` kontrolü **client-side** koda yazılırsa herkese açık olur.
- Kural: **güvenlik kararları (yetki, gizli veri) asla client-side'da olmaz; her zaman server-side.** Client'a gönderilen her satır, saldırgana verilmiş bir okuma iznidir.

## Kök sebep
- Güvenlik yine "gizleme" (security through obscurity) ile sağlanmaya çalışılmış; üstelik gizleme bile başarısız (adres JS'te).
- Doğrusu: panel, URL'i bilen herkese değil, server-side doğrulanmış yetkili role açık olmalı.
