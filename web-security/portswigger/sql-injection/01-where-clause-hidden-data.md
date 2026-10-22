# SQLi Lab #1: WHERE clause — gizli veriyi ortaya çıkarma — 22.10.2026

> İlk SQL Injection. Üçüncü büyük zafiyet sınıfı. XSS client-side'dı; bu server-side — uygulamanın database ile konuştuğu SQL cümlesine sızıyoruz.

**Lab:** PortSwigger → SQL injection → *SQL injection vulnerability in WHERE clause allowing retrieval of hidden data* (Apprentice)
**Görev:** Yayınlanmamış (unreleased/gizli) ürünleri de listeletecek bir SQLi gerçekleştir.

## Enjeksiyon noktasını bulma (recon)
- Önce `product?productId=9` denendi → bu tek bir ürün getiriyor (lookup), gizli bir liste yok. Yanlış nokta.
- Doğru nokta: kategoriye tıklayınca oluşan **`/filter?category=Lifestyle`** → bir koşulla süzülen bir LİSTE. Asıl av burası.

Tahmin edilen arka sorgu:
```sql
SELECT * FROM products WHERE category = 'Lifestyle' AND released = 1
```
`AND released = 1` → yayınlanmamış ürünleri senden saklayan şart.

## Çözüm
Payload (`category` değerine):
```
' OR 1=1 --
```
Sorgu şuna döndü:
```sql
SELECT * FROM products WHERE category = '' OR 1=1 --' AND released = 1
```
- `'` → uygulamanın açtığı tırnağı kapattı (**breakout**) → artık tırnağın dışında SQL kodu yazabiliyorum.
- `OR 1=1` → `1=1` her zaman doğru; `OR` yüzünden tüm satırlar eşleşti (gizli/unreleased dahil).
- `--` → arkadaki ` AND released = 1` ve uygulamanın kapanmamış tırnağını **yorum satırı** yapıp kesti → cümle geçerli kaldı → **Solved.** ✅

(Not: bu lab için sadece `'--` bile yeterdi — çünkü `released = 1` şartını kesmek gizli ürünleri zaten açar. `OR 1=1` filtreyi büsbütün genişletir; ikisi de çözer.)

## Takıldığı yer (öğretici hatalar)
1. İlk deneme `'OR1=1` → **Internal Server Error**. İki sebep:
   - `OR1=1` bitişik → SQL tek token sanıp anlamadı. Doğrusu boşluklu: `OR 1=1`.
   - Arkada kalan ` AND released = 1` + uygulamanın kapanmamış `'`'i cümleyi bozdu → syntax error.
2. **Ders:** 500 hatası aslında İYİ haber — girdinin SQL'e ulaşıp syntax'ı bozduğunu kanıtlar (enjeksiyon var). Eksik olan, cümleyi tekrar **geçerli** bırakmak. (XSS'teki "kaçmak yetmez, kalan kodu geçerli bırak" dersinin birebir aynısı.)

## 🛡️ Nasıl kapatılır (remediation)
Kök sebep: SQL cümlesi, kullanıcı girdisiyle **string birleştirilerek** kuruluyor → veri (data) ile komut (code) aynı metinde eriyor.

- **Asıl çözüm — parameterized queries / prepared statements:** Komutu ve değeri ayrı kanaldan gönder. Değerin konduğu yer bir "boşluk" (placeholder) olur; oraya yazılan her şey **her zaman veri** olarak işlenir, asla kod olarak değil.
  ```csharp
  // YANLIŞ — açık
  var sql = "SELECT * FROM products WHERE category = '" + category + "'";
  // DOĞRU — parameterized (ASP.NET / ADO.NET)
  var sql = "SELECT * FROM products WHERE category = @cat";
  cmd.Parameters.AddWithValue("@cat", category);
  ```
  EF Core'da `Where(p => p.Category == category)` zaten parameterize eder; `FromSqlRaw` ile string birleştirmek açığı geri açar. Dapper'da `new { cat = category }`.
- **Defense-in-depth (tek başına yetmez):**
  - Input validation / allowlist: `category` yalnızca bilinen değerlerden biri olabilir.
  - Least privilege: uygulamanın DB kullanıcısına gereğinden fazla yetki verme.
- **İlke:** Kullanıcı girdisi asla SQL cümlesinin metnine karışmaz; ayrı bir parametre olarak gider.

## Ders (genel çerçeve)
- SQLi'nin kalbi XSS ile aynı hastalık: **veri, kod olarak yorumlanıyor.** Tek fark dil (SQL) ve yer (server-side, DB).
- İki temel silah: `--` (kalanı kes) ve `' OR 1=1` (şartı her zaman doğru yap). Özü: **tırnaktan kaç, sonra SQL mantığını kendi lehine yeniden yaz.**
- Cure de XSS ile aynı aile: "veriyi koddan ayır" — XSS'te output encode, SQLi'de parameterization.

## Zincir bağı
- XSS'te context'ten `"`/`'` ile kaçıyorduk; burada da `'` ile SQL string'inden kaçtık — aynı break-out kası, yeni dil.
- "500 hatası = enjeksiyon kanıtı" fikri, ileride blind/error-based SQLi'nin de temeli olacak.
