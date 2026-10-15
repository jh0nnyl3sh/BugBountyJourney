# 13. XSS Lab #1: Reflected XSS — HTML context, nothing encoded — 15.10.2026

> Zincire yeni halka: XSS. Artık bir şeye erişmiyoruz; siteye kendi kodumuzu enjekte edip çalıştırıyoruz.

**Lab:** PortSwigger → Cross-site scripting → Reflected → *Reflected XSS into HTML context with nothing encoded* (Apprentice)
**Görev:** `alert` fonksiyonunu çalıştıran bir XSS gerçekleştir.

## Çözüm
1. Arama (search) kutusuna zararsız bir kelime (`ugurtest123`) yazıldı, arandı.
2. View Page Source → kelime, sayfanın **HTML gövdesine düz metin** olarak basılmış (hazır bir JS'in içine değil).
3. Arama kutusuna `<script>alert(1)</script>` yazıldı → tarayıcı bunu **yeni bir script etiketi** olarak işledi ve çalıştırdı → uyarı çıktı → **Solved.** ✅

## Kritik kavramlar
- **HTML context:** Girdi, HTML gövdesine basılıyor. O yüzden ham `<script>...</script>` enjekte etmek işe yarar (yeni etiket eklemiş oluyoruz). İleride girdi *var olan bir JS'in içine* düşerse `<script>` işe yaramaz, farklı yöntem gerekir (context önemli!).
- **"nothing encoded":** Site `<` ve `>` karakterlerini dönüştürmeden basıyor. Güvenli site `<`'yi `&lt;` yapardı → o zaman `<script>` bir etiket değil, ekranda düz yazı olurdu. XSS'in kalbi: girdinin encode/escape edilip edilmemesi.

## XSS'in access control'den farkı (bocalama buradan öğretti)
- Access control'de genelde **yanıtı/isteği** kurcalıyorduk (Admin=false → true).
- XSS'te saldırı **girdide (input)** olur: zararlı kodu bir girdi alanına (arama kutusu) koyarsın, sunucu onu **kendisi** yanıta gömer.
- Burp'te yanıtı elle değiştirmek işe yaramaz; o sadece *senin ekranındaki* kopyayı değiştirir. Gerçek XSS, sunucunun **herkese** o zararlı HTML'i üretmesini sağlamaktır.

## Kök sebep
- Sunucu, kullanıcı girdisini "veri" değil "kod" olarak işliyor (encode/sanitize etmeden HTML'e gömüyor).
- Doğrusu: kullanıcıdan gelen her girdi, ekrana basılmadan önce bağlamına uygun şekilde encode edilmeli.

## Not — kendi projesine bağ
Kendi uygulamasındaki arama kutusu da tam bu türden bir yansıma noktası. XSS'i artık o gözle görecek: "ben kullanıcı girdisini ekrana basarken encode ediyor muyum?"
