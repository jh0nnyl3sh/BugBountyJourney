# XSS Lab #5: DOM-based — document.write sink, source: location.search — 18.10.2026

> İlk DOM-based XSS. Önceki lab'larda zararlı kodu **sunucu** yanıta gömüyordu; burada sunucu masum — zararı yapan tarayıcının kendi JavaScript'i. Saldırı tamamen client-side.

**Lab:** PortSwigger → XSS → DOM-based → *DOM XSS in document.write sink using source location.search* (Apprentice)
**Görev:** `alert` çağıran bir DOM XSS gerçekleştir.

## Yeni kavram: source → sink (kaynak → lavabo)
Fabrika bandı gibi düşün:
- **Source (kaynak):** Saldırganın kontrol ettiği girdinin banda GİRDİĞİ yer. Burada `location.search` (URL'deki `?search=...` kısmı).
- **Sink (lavabo):** O girdinin tehlikeli biçimde KULLANILDIĞI yer. Burada `document.write(...)` — kendisine verilen metni ham HTML olarak sayfaya yazar.
- Girdi, source'tan sink'e **hiç temizlenmeden** akıyorsa → DOM XSS.

İlgili JS (sayfanın kendi kodu):
```js
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) { trackSearch(query); }
```

## Keşif — girdi İKİ yere gidiyor (kritik)
1. **h1 içinde** ("0 search results for 'X'"): sunucudan geliyor, `<` `>` **encode ediliyor** → ölü tuzak, sömürülemez.
2. **`document.write`'ın ürettiği `<img src="...">` içinde**: client-side yazılıyor, `<` `>` **encode edilmiyor** → asıl av burası.

**DOM XSS püf noktası:** `document.write` sayfa yüklenirken SONRADAN çalışır; ürettiği `<img>`, `View Page Source`'ta GÖRÜNMEZ. Gerçek yeri görmek için **DevTools → Elements (canlı DOM)** gerekir. Kural: *View Source ≠ canlı DOM.*

## Çözüm
Payload (search box'a ya da `?search=` parametresine):
```
"><svg onload=alert(1)>
```
Adım adım:
- Girdi şuraya düşüyor: `<img src="/resources/.../tracker.gif?searchTerms=GİRDİ">`
- `"` → `src` özniteliğini kapatır (attribute **çift tırnaklı**, o yüzden çift tırnak şart).
- `>` → `<img>` etiketini erkenden kapatır.
- `<svg onload=alert(1)>` → yepyeni bir etiket; yüklenince `onload` tetiklenir, `alert(1)` çalışır → **Solved.** ✅

Sonuç (canlı DOM'da):
```html
<img src="/resources/.../tracker.gif?searchTerms="><svg onload=alert(1)>">
```

## Takıldığı yer (öğretici hatalar)
Saatlerce kapı açılmadı; iki sebep vardı:
1. **Yanlış tırnak:** `'><svg...` (tek tırnak) denendi. Ama `src="..."` çift tırnaklı → tek tırnak attribute'u KAPATMAZ, break out olmaz. Doğrusu `"`.
2. **alert çağrılmadı:** `onload='hackedbyj0hnyl3sh'` / `onload='1'` → bunlar onload'a sadece METİN atıyor, hiçbir şey ÇALIŞTIRMIYOR. Gerçek JS için `onload=alert(1)` gerekir.
3. **Yanlış reflection'a bakıldı:** encode'lu h1'e odaklanıldı; asıl encode'suz nokta (`document.write`/img src) gözden kaçtı.

## 🛡️ Nasıl kapatılır (remediation)
Kök sebep: güvenilmeyen girdi (`location.search`) temizlenmeden tehlikeli bir sink'e (`document.write`) akıyor.

- **Tehlikeli sink'ten kaçın:** `document.write` + string birleştirme yerine güvenli DOM API'leri kullan. Metin basacaksan `element.textContent = query` (HTML olarak değil, düz metin olarak yazar; `<` `>` otomatik etkisizleşir).
- **Mecbursan context'e uygun encode et:** Girdiyi HTML'e koyacaksan HTML-encode, bir attribute'a koyacaksan attribute-encode et. "Her girdi kod değil veridir" kuralı.
- **Kütüphane:** HTML üretmek zorunluysa **DOMPurify** gibi bir sanitizer'dan geçir (`DOMPurify.sanitize(query)`).
- **Savunma katmanı (defense-in-depth):** Katı bir **Content-Security-Policy** (ör. inline script/`unsafe-inline` yok) XSS'in etkisini sınırlar — ama tek başına çözüm değil, asıl düzeltme girdiyi sink'e sokmamaktır.
- **Genel ilke:** Bu bir CLIENT-SIDE açık; sunucu tarafı encode burada KORUMAZ. Savunma, kodun çalıştığı yerde (tarayıcıda, JS'in kendisinde) olmalı.

## Ders (DOM XSS genel kural)
- DOM XSS'i avlamak = **source → sink** zincirini takip etmek. Hangi source'lar kontrol edilebilir (`location`, `document.referrer`, `postMessage`...), hangi sink'ler tehlikeli (`document.write`, `innerHTML`, `eval`, `setTimeout(string)`...) — bu haritayı ezberlemeden, mantığıyla tanı.
- Sunucu yanıtı masum görünse bile zafiyet client JS'inde olabilir → her zaman canlı DOM'a bak.

## Zincir bağı
- Lab #4'te bir JS string'inden `'` ile kaçmıştık. Burada bir HTML attribute'undan `"` ile kaçtık — aynı break-out kası, sınır karakteri context'e göre değişti.
- Fark: #4 sunucu-kaynaklı (reflected) idi; bu tamamen client-side (DOM). "Zafiyet nerede doğuyor?" sorusunun cevabı değişti.
