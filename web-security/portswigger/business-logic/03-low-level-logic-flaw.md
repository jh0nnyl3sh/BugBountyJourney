# Business Logic #3: Low-level logic flaw (integer overflow)

- **Seviye:** Practitioner
- **Sınıf:** Business Logic — integer overflow
- **Yöntem:** Körleme + Python otomasyon + modüler matematik
- **Araç:** Python requests (iki gün süren plumbing mücadelesi dahil 😅)

---

## Zafiyet

Sepet toplamı sunucuda **signed 32-bit integer** olarak tutuluyor (üst sınır ~2,147,483,647).
Yeterince ürün eklersen toplam bu sınırı **aşar** ve **negatife/düşük bir değere sarar**
(integer overflow). Bu sarmayı kullanıp toplamı **bakiyeye sığan** küçük bir pozitife indirdik.

### Çocuğa anlatır gibi
9999'a kadar sayan bir sayaç: 9999'dan sonra bir daha artırınca **0000'a** döner. Burada da
toplam, maksimumu geçince başa/negatife sarıyor.

## Kök neden

> Sunucu, fiyatı **signed 32-bit int** gibi sınırlı bir tipte tutuyor ve **taşmaya karşı kontrol
> yok.** Toplam int sınırını aşınca sarıyor; uygulama bu "imkansız" durumu hesaba katmamış.

## Çözüm — ve asıl ders: "kaba otomasyon ≠ çözüm, KESİNLİK çözüm"

### Yanlış giden (watch-and-chase)
İlk yaklaşım: qty-99 ceket ekleyip **toplamı izle, sweet spot'ta yakala.** Sorun: her qty-99 adımı
~**$130k** atlıyor, hedef pencere ise **dar** (~$100). Adım pencereden büyük → sürekli üstünden
atladık, negatiften 0'a çıkarken $100 altını tutturamadık. **Otomatikti ama kaba.**

### Doğru giden (compute-exact)
Bu bir **modüler aritmetik** problemi: öyle bir N (ceket adedi) var ki `N × fiyat mod 2³²` tam
bakiyeye sığar. İzleyip yakalamak yerine **o N'i matematikle hesapla**, tam o kadar ekle:
```python
delta = 0
while not (0 < (T + delta * P) % (2**32) <= CREDIT):
    delta += 1
# delta ceketi ekle (qty99 batch + kalan) -> toplam tam oturur
```
**Çözüm otomasyon değil, kesinlikti.** Deneme-yanılma yerine doğrudan hedefi hesapladık.

## Payload / akış
1. Ceketi sepete koy (zaten almak istediğimiz).
2. Python ile mevcut toplamı oku → `delta`'yı hesapla → o kadar ceket ekle (qty-99 batch).
3. Toplam `(0, $100]` penceresine oturunca → tarayıcıda **Checkout** → çözüldü.

## İki günlük plumbing dersleri (sahada da olacak)

Bu lab'ın **konsepti** 5 dakikada oturdu; zor olan **çalıştırmaktı.** Takıldığımız gerçek-dünya tuzakları:
- **URL path doubling (imza hatam):** base URL sonunda `/` → `URL + "/cart"` = `...//cart` → 404.
  **Refleks: base URL asla `/` ile bitmez.** (3+ kez tekrarladı.)
- **Bayat session:** cookie süresi dolunca `/cart` login'e döner → total okunamaz (None).
  Taze session şart; lab restart olunca URL de session de değişir.
- **Regex kırılganlığı:** "Total:" ile fiyat arasında HTML etiketleri var → `Total:.*?\$...` + `re.S`.
- **Debugging refleksi:** None gelince tahmin etme → status + HTML yazdır, neyle karşı karşıyasın GÖR.

> **Sahanın dersi:** Bir bug'ı anlamak dakikalar, onu güvenilir çalıştırmak saatler/günler sürebilir.
> Dayanıklılık, zafiyet bilgisinden daha belirleyici.

## Remediation (defender)

1. **Taşmaya dayanıklı tip / kontrol:** fiyat/toplam için 64-bit veya sınırsız (decimal/BigInteger)
   tip; ya da her işlemde üst/alt sınır kontrolü.
2. **Makul sınır (business rule):** sepet toplamı/adedi için mantıklı bir üst limit (kimse 16.000
   ceket almaz) → anomaliyi reddet.
3. **Sunucu tarafı doğrulama:** toplam, miktar, bakiye — hepsi güvenilir kaynaktan, taşma kontrollü.

## Büyük resim bağı
- BL #1: sunucu **price**'a güvendi.
- BL #2: **negatif quantity** (alt sınır ihlali).
- BL #3: **integer overflow** (üst sınır ihlali) → aynı aile: uygulamanın "makul değer" varsayımını,
  bu sefer **tipin sınırını** zorlayarak kır.

## Köşe taşı

> Her sayısal alanın bir **tip sınırı** vardır. O sınırı aşmak (overflow) "imkansız" sanılan bir
> durum yaratır ve uygulama onu hesaba katmamışsa sömürülür. Ve: kaba otomasyonla yakalamaya
> çalışma — hedefi **hesapla**, tam otur.
