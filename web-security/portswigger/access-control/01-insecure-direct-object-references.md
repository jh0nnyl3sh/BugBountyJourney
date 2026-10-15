# 6. İlk Çözülen Lab: IDOR (PortSwigger) — 09.10.2026 🎉

> İlk gerçek zafiyet avı. Teoride bilinen IDOR, ilk kez elle bulunup sömürüldü.

**Lab:** PortSwigger Web Security Academy → Access control → *Insecure direct object references* (Apprentice)
**Görev:** `carlos` kullanıcısının şifresini bul ve hesabına giriş yap.
**Araç:** Firefox + FoxyProxy + Burp Suite (HTTP history).

## Lab'ın ipucu (açıklama cümlesi)
> "This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs."
→ Sohbet kayıtları diskte tutuluyor, sabit URL'lerle getiriliyor. Av burada.

## Keşif / eleme (avcı mantığı)
- Ana sayfa: ürünler, ID'ler sıralı ama **herkese açık** → IDOR yok (sıralı ID her zaman IDOR değildir).
- My account: sadece login, **kayıt yok** → o kapı kapalı.
- **Live chat:** sohbet + transcript'i `.txt` olarak **indirme** seçeneği → ilgi çekici olan bu.

## Sömürü adımları
1. Live chat'e girip birkaç mesaj yazıldı.
2. Transcript indirme linkine bakıldı; Burp **HTTP history**'de istek şuydu:
   ```
   GET /download-transcript/4.txt
   ```
   (4 = benim transcript'im)
3. **Avcı refleksi:** URL'deki sayı doğrudan bir dosyaya işaret ediyor → değiştirirsem?
4. Sistematik deneme: 5 (boş), 0 (boş), **1 → carlos'un sohbeti açıldı.**
5. Transcript içinde carlos'un **şifresi** düz metin olarak geçiyordu.
6. O şifreyle `carlos` olarak login → **Solved.** ✅

## Teorik karşılığı
- **Nesne (object):** sunucudaki bir dosya (`1.txt`).
- **Doğrudan başvuru:** sabit, tahmin edilebilir URL (`/download-transcript/<n>.txt`).
- **Kök sebep:** sunucu "bu dosya senin mi?" diye **authorization** kontrolü yapmıyor.
- **Tür:** yatay yetki yükseltme (horizontal privilege escalation) — aynı seviyedeki başka kullanıcının verisine geçiş.

## Çıkarılan dersler
- Önce **trafiği izle** (Burp HTTP history), sonra saldır. Siteye dalıp Burp'ü unutmamak lazım.
- Sıralı/okunur bir ID gördüğünde dur ve sor: bu başkasına ait, görmemem gereken bir şeye mi işaret ediyor?
- Deneme **sistematik** olmalı (0'dan tara), rastgele değil → ileride script'le otomatikleştirilecek mantığın elle hali.
