# 10. Access Control Lab #5: Unprotected admin functionality — 12.10.2026

> Access control'ün IDOR'dan farklı yüzü: başkasının verisine değil, **erişmemen gereken bir işleve** ulaşmak. İlk content discovery denemesi.

**Lab:** PortSwigger → Access control → *Unprotected admin functionality* (Apprentice)
**Görev:** Admin paneline eriş ve `carlos` kullanıcısını sil.

## Çözüm
1. Sitenin kökünde `/robots.txt` dosyasına bakıldı.
2. İçindeki `Disallow:` satırı admin panelinin yolunu veriyordu: `/administrator-panel`.
3. O yola gidildi → admin paneli **hiçbir yetki kontrolü olmadan** açıldı.
4. Panelden carlos silindi → **Solved.** ✅

## robots.txt nedir?
- Arama motoru botlarına "şurayı tarama" demek için sitenin kökünde duran standart dosya.
- İroni: site sahibi "gizli" yolları `Disallow` listesine yazar, ama dosya herkese açıktır → gizlemek isterken adres tarif etmiş olur.
- Avcı için ilk bakılacak yerlerden biri.

## Kök sebep
- Admin işlevi sadece **"gizlenerek"** korunmuş (security through obscurity), gerçek bir yetki kontrolü yok.
- Doğrusu: hassas işlev, URL'i bilen herkese değil, **sadece yetkili role** açık olmalı (sunucu tarafında kontrol).

## Daha geniş ders — content discovery (içerik/dizin keşfi)
`robots.txt` bu ailenin tek üyesi değil. Gizli yol/işlev bulmanın başka yolları:
- JavaScript dosyalarında unutulmuş endpoint'ler
- HTML yorum satırlarındaki ipuçları
- Hata sayfalarının sızdırdığı bilgiler
- **Wordlist ile otomatik tarama** (ffuf, gobuster, dirb) → binlerce olası yolu (`/admin`, `/backup`, `/old`...) otomatik dener
→ İleride: önce elle anla, sonra Python/otomasyon ile ölçekle.

## Süreç notu
Önce HTTP history + Intercept ile ipucu arandı, Google'da arama yapıldı; robots.txt bilinmediği için son çare soruldu. Lab'a yine "körleme" (açıklama okumadan) girildi. Süreç bilinçli olarak aktif tutuluyor.
