# 8. IDOR Lab #3: Unpredictable user IDs — 11.10.2026

> Üçüncü çözülen lab. "Karmaşık ID güvenli demek değil; bir yerde sızıyor olabilir" dersinin sahada kanıtı.

**Lab:** PortSwigger → Access control → *User ID controlled by request parameter, with unpredictable user IDs* (Apprentice)
**Görev:** `carlos`'un API key'ini bul ve gönder.
**Verilen hesap:** `wiener:peter`

## Dünkü labdan farkı
- Dün: `id=wiener` → `id=carlos` (okunur, basit).
- Bugün: `id=f043469e-....` gibi **GUID** (tahmin edilemez). "carlos yaz geç" işlemez.
- Yani önce **carlos'un GUID'ini bulmak** gerekiyor.

## Kullanılan yöntem (not: süreç, sonuçtan önemli)
1. GUID'i görünce önce **hipotez:** "bu bir hash olabilir mi?" → Google'da araştırıldı (AI'ya çözdürmeden), hash kapısı elendi.
2. "Bir kullanıcının kimliği başkalarına **nerede** gösterilir?" sorusu → chat yok, ama **blog yorumları** var; yorumlarda carlos'un adı görünüyor.
3. **Doğrulama adımı (kritik):** önce *kendi* yorumuna tıklandı, URL'de çıkan ID ile kendi ID'si eşleştirildi → "yorumdaki isme tıklayınca o kişinin ID'si URL'e düşüyor" mekanizması kendi üstünde doğrulandı.
4. Aynısı carlos'a: carlos'un yorumundaki ismine tıkla → carlos'un GUID'i alındı.
5. `/my-account?id=<carlos-guid>` → carlos'un hesabı, API key alındı → gönderildi → **Solved.** ✅

## Teorik karşılığı
- Kök sebep yine **authorization eksikliği** (sunucu "bu hesap senin mi?" diye sormuyor).
- Ek ders: **tahmin edilemezlik ≠ yetkilendirme.** Karmaşık ID sadece "bulmayı" zorlaştırır; ID bir yerde (burada yorum yazarı bilgisinde) **sızınca** koruma çöker.

## Çıkarılan dersler
- Hazıra konma; önce kendi hipotezini kur ve test et. Cevabı almak kas oluşturmaz, bulmak oluşturur.
- Bir mekanizmayı **saldırmadan önce kendi üstünde doğrula** (kendi ID'ni eşleştir), sonra hedefe uygula.
- "Kimlik başkalarına nerede gösterilir?" sorusu, sızıntı noktalarını bulmanın anahtarı (profil, yorum, yazar, paylaşım linki...).
