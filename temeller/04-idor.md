# 4. IDOR (Insecure Direct Object Reference)

> İsmi unut, mantığı tut: "Güvensiz doğrudan nesne başvurusu."

## Mantık

Profil sayfanda adres şu olsun:

```
example.com/profil?id=1004
```

Sen 1004'sün, kendi bilgilerini görüyorsun. Avcı sorusu: **Ya 1004'ü silip 1005 yazarsam?**

- **Sağlam site:** "Sen 1004'sün ama 1005'in verisini istiyorsun — bu senin değil" der, durdurur. ✅
- **Güvensiz site:** numaraya bakar, 1005'i çeker, sana getirir. Hiç sormadan. → **IDOR.**

## Kök sebep: Yetki kontrolünün (authorization) yapılmaması

İki farklı şey var, karıştırma:

- **Authentication (kimlik doğrulama):** "Sen kimsin?" Giriş yaptın mı? Session ID'n geçerli mi?
- **Authorization (yetkilendirme):** "Peki sen bunu görmeye yetkili misin?"

IDOR = kimliğin **gerçek**, ama erişimin **kaçak**. Garson bilekliğine bakıp seni içeri alıyor, ama mutfağa dalıp başkasının siparişini aldığında kimse "dur o senin değil" demiyor.

## Sinsi kardeşi — "tahmin edilemez" sanılan ID'ler

```
example.com/fatura?id=8f14e45fceea167a5a36dedd4bea2543
```

Böyle karmaşık ID görünce "burada iş yok" **deme**.

- **Tahmin edilemezlik ≠ yetkilendirme.** Kapı kilitli değil, sadece numarası okunması zor yazılmış.
- O karmaşık değer aslında sıralı bir ID'nin **hash'i** olabilir (örn. `1004`'ün MD5'i), bir yerde **sızıyor** olabilir (başka yanıt, e-posta, paylaşım linki), ya da yetki **hiç kontrol edilmiyordur**.
- Asıl ödüller, çoğunluğun "karmaşık, geçeyim" dediği yerde saklı. **Sen geçme, sor.**

## Yanıtı okuma inceliği

Yanıtın *ne* olduğu kadar *nasıl* olduğu da ipucudur:

- `403 Forbidden` → bilerek durduruyor (iyi)
- `404 Not Found` → "o kayıt yok" gibi davranıyor (bazen bilgi sızdırır)
- `200 OK` ama içi boş → ayrı bir sinyal

## Akraba teknik (karıştırma)

`/admin`, `/users` gibi gizli sayfa denemeleri IDOR **değil** → bu **içerik/dizin keşfi** (content discovery). Ayrı kas, ileride ayrıca işlenecek.

- IDOR = var olan bir nesnenin ID'sini değiştirmek.
- İçerik keşfi = gizli/korunmasız sayfaları bulmak.

## Neden önemli

Bug bounty'de en sık bulunan ve en çok ödül kazandıran zafiyetlerden biri: bulması görece kolay, etkisi büyük (binlerce kullanıcının verisi sızabilir). Otomasyonla (bir script'le binlerce ID denemek) çok iyi örtüşür — ama **sadece scope içinde veya kendi lab'ında**.
