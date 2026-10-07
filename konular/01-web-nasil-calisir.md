# 1. Web Nasıl Çalışır? — İstek/Yanıt Döngüsü

> Bir web zafiyeti, bu sürecin bir yerindeki çatlakta yaşar. Süreci bilmezsen çatlağı göremezsin.

Tarayıcıya adres yazıp Enter'a basınca sırayla şunlar olur:

1. **DNS** — alan adını IP'ye çevirir (internetin telefon rehberi). Bilgisayarlar isimle değil numarayla konuşur.
2. **Bağlantı** — TCP three-way handshake (3'lü el sıkışma): "bağlanmak istiyorum" → "tamam bağlan" → "bağlandım". HTTPS bunun üstüne şifreli bir zarf ekler (kimse arada dinleyemesin diye).
3. **HTTP İsteği (Request)** — tarayıcı sunucuya "bana şunu gönder" der. → *Avcılığın kalbi burada; bu isteği durdurup, bakıp, değiştireceğiz.*
4. **HTTP Yanıtı (Response)** — sunucu isteği işler, belki veritabanına bakar, sonra HTML + durum kodu + ek bilgiler döner.
5. **Çizim** — tarayıcı gelen HTML/CSS/JS'i alıp sayfayı ekrana çizer.

**Özü:** Bütün web şu döngüde döner → **İstek git, yanıt gel.** Milyarlarca kez, her saniye.
