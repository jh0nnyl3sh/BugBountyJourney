# XSS Lab #6: DOM-based — innerHTML sink, source: location.search — 20.10.2026

> İkinci DOM-based lab. Source aynı (`location.search`), sink değişti (`innerHTML`). Asıl ders: sink'in TÜRÜ bağlamı değiştirir — ve innerHTML'in kendine özgü bir "çalışmaz" tuzağı var.

**Lab:** PortSwigger → XSS → DOM-based → *DOM XSS in innerHTML sink using source location.search* (Apprentice)
**Görev:** `alert` çağıran bir DOM XSS gerçekleştir.

## İlgili JS (sayfanın kendi kodu)
```js
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) { doSearchQuery(query); }
```
Girdi, bir `<span id="searchMessage">` elementinin **innerHTML**'ine atanıyor. Yapı şöyle:
```html
<h1>0 search results for '<span id="searchMessage">GİRDİ</span>'</h1>
```

## Çözüm
Payload (search box'a ya da `?search=` parametresine):
```
<img src=errorpls onerror=alert(document.domain)>
```
(PortSwigger'ın resmi çözümü de aynı mantık: `<img src=1 onerror=alert(1)>`)

## Neden breakout YOK?
- `document.write` lab'ında girdi bir **attribute'un** içine düşüyordu (`<img src="...GİRDİ...">`) → `"` ile tırnaktan, `>` ile tag'den kaçmak gerekti.
- Burada girdi doğrudan **HTML içeriği** olarak atanıyor (`span.innerHTML = query`) → zaten HTML içeriği bağlamındasın. Kaçacak tırnak, kıracak attribute yok. Ne HTML yazarsan render edilir.
- h1'deki `'...'` tırnakları bir JS string/attribute değil, ekrandaki **düz metin** → kapatılacak bir şey değil (buraya takılmak, önceki lab'ların refleksini yanlış bağlama taşımaktı).

## Neden `<img onerror>` — asıl mekanizma (innerHTML tuzağı)
- **innerHTML ile sokulan `<script>` ÇALIŞMAZ** (HTML spec kuralı).
- `<svg onload>` / `<body onload>` de çalışmaz: `onload` sayfa ilk yüklenirken tetiklenir; sayfa yüklendikten sonra innerHTML ile eklenen elementte tekrar tetiklenmez.
- `<img src=errorpls onerror=...>` çalışır çünkü: tarayıcı var olmayan `src`'yi yüklemeye çalışır → **başarısız olur** → `onerror` **anında, kullanıcı müdahalesi olmadan** tetiklenir. onerror, sonradan eklenen elementte de ateşlenir.
- **Analoji:** `onerror` bir tuzak teli. Var olmayan bir resme işaret eden `<img>` koyarsın; tarayıcı getirmeye çalışırken tele takılır, "başarısız olursan şunu yap" komutun patlar. Kimseyi beklemez. (`onclick`/`onmouseover` bir insanın müdahalesini bekler → otomatik çalışmaz.)

## Takıldığı yer (öğretici hatalar)
1. `'alert(1)` denendi → innerHTML'de kapatılacak tırnak yok; breakout refleksi yanlış bağlama taşındı.
2. `onmouseclick="alert(1)"` düz metin olarak yazıldı → `onmouseclick` diye bir event yok (doğrusu `onclick`), üstelik bir element'in attribute'u değil düz metindi, hem de kullanıcı müdahalesi gerektirirdi → hiçbir şey olmadı.
3. h1'deki `'` kapatılmaya çalışıldı (red herring).
4. View Page Source'ta girdi aranıp bulunamadı → DOM XSS canlı DOM'da görünür (Elements), View Source'ta değil.

## 🛡️ Nasıl kapatılır (remediation)
Kök sebep: güvenilmeyen girdi (`location.search`) temizlenmeden `innerHTML` sink'ine akıyor.

- **Güvenli sink kullan:** HTML değil düz metin basacaksan `element.textContent = query` (innerHTML yerine). textContent, `<img>`'yi etiket değil düz yazı olarak basar → kod çalışmaz.
- **HTML üretmek zorunluysa sanitize et:** `DOMPurify.sanitize(query)` gibi bir kütüphaneden geçir; `onerror` gibi tehlikeli attribute'lar temizlenir.
- **Context'e uygun encode:** Girdi HTML'e girecekse HTML-encode et ("veri, kod değil").
- **Defense-in-depth:** Katı bir Content-Security-Policy (inline event handler/`unsafe-inline` yok) etkiyi sınırlar — ama tek başına çözüm değil; asıl düzeltme girdiyi tehlikeli sink'e sokmamak.
- **İlke:** Client-side açık; sunucu-tarafı encode burada KORUMAZ. Savunma, kodun çalıştığı yerde (tarayıcıda, JS'in kendisinde) olmalı.

## Ders (genel kural)
- Aynı source, farklı sink → farklı context → farklı teknik. Context'i "tırnak var mı" diye değil, "girdim tam olarak nasıl kullanılıyor (attribute mı, HTML içeriği mi, JS string mi)" diye oku.
- innerHTML sink'te: breakout yok → doğrudan HTML sok → `<script>`/`onload` çalışmaz → **otomatik tetiklenen event** lazım → `<img src=x onerror=...>` klasik çözüm.

## Zincir bağı
- Lab #5 (document.write): attribute'tan `"` ile breakout.
- Lab #6 (innerHTML): breakout yok, ama innerHTML'in çalıştırma kuralları yüzünden event seçimi (`onerror`) öne çıktı.
- Ortak iskelet: source → sink takibi + canlı DOM. Değişen: sink'in tipi ve ona uygun tetikleme.
