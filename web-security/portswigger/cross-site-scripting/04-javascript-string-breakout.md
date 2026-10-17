# XSS Lab #4: Reflected — JavaScript string, angle brackets HTML-encoded — 17.10.2026

> İkinci "context" lab'ı. Bu kez girdi var olan bir JavaScript string'inin içine düşüyor; oradan kaçıp JS'i geçerli bırakmak gerekiyor.

**Lab:** PortSwigger → XSS → Contexts → *Reflected XSS into a JavaScript string with angle brackets HTML encoded* (Apprentice)
**Görev:** JavaScript string'inden break out et ve `alert` çağır.

## Keşif
1. Arama kutusuna `checkthebox000001` yazıldı.
2. View Page Source → girdi bir `<script>` bloğunda, bir JS string'inin içinde:
   ```html
   <script> var searchTerms = 'checkthebox000001'; </script>
   ```
3. `<` `>` encode ediliyor → HTML'e sıçrayıp etiket açmak yok. Ama zaten `<script>` içindeyiz; hedef HTML değil, içinde bulunduğumuz **string**.

## Çözüm
Payload (search box'a): `'-alert(1)-'`

Sonuç:
```js
var searchTerms = ''-alert(1)-'';
```
- Baştaki `'` → string'i kapatır (break out).
- `-alert(1)-` → `alert(1)`'i bir ifade (expression) olarak çalıştırır (çıkarma zincirinin ortasında değerlendirilir).
- Sondaki `'` → geriye kalan `';` ile eşleşir, JS **geçerli (valid)** kalır → syntax error yok → çalışır → **Solved.** ✅

## Takıldığı yer (öğretici hata)
Denenen: `'-alert(hacked by j0hnyl3sh)-'` → **çalışmadı.**
Sebep: `alert(...)`'ın içindeki `hacked by j0hnyl3sh` tırnaksız, boşluklu kelimeler → bu geçerli bir JavaScript değil (syntax error). JS bunu string değil, kod olarak okur; tanımsız değişkenler + boşluklar hatayı patlatır.
- Metin geçirmek istersen tırnak/backtick gerekir ama attribute/string context'te tırnak çakışabilir.
- İspat için en temizi `alert(1)` ya da `alert(document.domain)` — argüman sade ve geçerli.

## Ders (JS context için genel kural)
String'den kaçmak yetmez; kaçtıktan sonra **kalan JS'i geçerli bırakmak** şart. İki yol:
- `';alert(1)//` → `;` ile bitir, `//` ile kalanı yoruma al.
- `'-alert(1)-'` → `-...-` ile ifadeyi araya sok, son `'` ile kalanı yut (daha zarif).

## Zincir bağı
- HTML attribute lab'ında `"` ile kutudan kaçtık. Burada `'` ile string'den kaçtık. Aynı break-out kası, farklı sınır karakteri ve "geçerli bırak" şartı.
