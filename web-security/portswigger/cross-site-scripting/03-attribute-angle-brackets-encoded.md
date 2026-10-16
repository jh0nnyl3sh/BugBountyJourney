# XSS Lab #3: Reflected — attribute context, angle brackets HTML-encoded — 16.10.2026

> İlk "context" lab'ı. Ders: payload ezberlemek değil, **girdinin hangi bağlama düştüğünü okuyup** ona göre silah seçmek.

**Lab:** PortSwigger → XSS → Contexts → *Reflected XSS into attribute with angle brackets HTML-encoded* (Apprentice)
**Görev:** Bir attribute enjekte edip `alert` çalıştır.

## Takılma yolu (bilerek debelendim)
1. HTML body refleksiyle `<script>alert(...)</script>` denendi → işe yaramadı.
2. Kaynak kodda `<` → `&lt;`, `>` → `&gt;` olmuş (angle brackets encode ediliyor) → yeni etiket yazılamıyor.
3. "Madem encode hali `&lt;`, direkt onu yazayım" denendi → **olmadı.** Çünkü encode, savunmanın kendisi: `&lt;` ekranda `<` *görünür* ama etiket olarak *işlenmez*. Entity yazarak encode geri alınamaz.

## Çözüm (break out + event handler)
Girdi bir özniteliğin içine düşüyordu: `<input ... value="GİRDİ">`.

Payload (search box'a): `"onmouseover="alert(1)`

Sonuç:
```html
<input ... value="" onmouseover="alert(1)">
```
- Baştaki `"` → `value` özniteliğini erkenden kapatır (kutudan **break out**).
- ` onmouseover="alert(1)` → **yeni bir event handler** ekler (JS'i `<script>` olmadan çalıştırır).
- Fare elementin üstüne gelince `alert(1)` çalışır → **Solved.** ✅

## Asıl bug nerede
- Site `<` ve `>`'yi encode etti ama **çift tırnağı (`"`) encode etmeyi unuttu.**
- `"` de `&quot;` yapılsaydı attribute'tan çıkılamaz, XSS olmazdı. Savunma **yarım** yapılmış.
- Doğru savunma: bağlama göre **tüm** tehlikeli karakterleri encode etmek.

## Context kavramı (bu lab'ın özü)
Girdi nereye düşüyor? Payload ona göre değişir:
| Context | Örnek | Silah |
|---------|-------|-------|
| HTML body | `<h1>GİRDİ</h1>` | ham `<script>` (lab #1, #2) |
| HTML attribute | `value="GİRDİ"` | `"` ile kaç + event handler (bu lab) |
| JavaScript içi | `var x='GİRDİ'` | string'den/JS'den kaçış (sonraki) |

**Yöntem:** Önce "hangi context?" diye sor, sonra silahı seç. Payload ezberleme, context oku.

## Yeni silahlar (cepte)
- **Event handler'lar:** `onmouseover`, `onclick`, `onfocus`, `onload`... JS'i `<script>` olmadan çalıştırır.
- **Break out:** bulunduğun yapıdan (attribute, string) uygun karakterle (`"`, `'`) çıkmak.
