# XSS Lab #9: Most tags and attributes blocked (filter bypass + iframe delivery)

- **Seviye:** Practitioner (ikinci filter bypass, bir tık daha zor)
- **Sınıf:** Cross-Site Scripting — filter/WAF bypass + delivery tekniği
- **Yöntem:** Körleme + Python enumerate (iki aşamalı) + iframe/resize delivery
- **Araç:** Python requests + exploit server

---

## İki yeni ders

### Ders 1 — "Enumeration, wordlist'in kadar iyidir" 🔑 (sahada en kritik)

İlk tarama sadece `desc` verdi — çünkü dünkü **SVG ağırlıklı küçük tag listesini** kullandım.
Liste eksikti → **asıl izinli tag (`body`) listede yoktu** → kaçırdım.

> **Eksik wordlist = false negative.** Gerçek hedefte zayıf listeyle tararsan var olan açığı
> göremezsin ve "burada bir şey yok" diye yanlış sonuca varırsın. **Bir şey bulamamak, orada
> bir şey olmadığını kanıtlamaz — sadece listen zayıftı.**

Kapsamlı HTML tag listesiyle (PortSwigger XSS cheat sheet) yeniden tarayınca **`body`** çıktı.
Aynısı event için: küçük event listende `onresize` yoktu; geniş listeyle tarayınca çıktı.

### Ders 2 — "Otomatik ateşlenen event vs dış tetik gerektiren event"

- **Dün (XSS #8):** `<svg><animatetransform onbegin=...>` → `onbegin` **kendiliğinden** ateşlendi
  (SVG animasyonu sayfa açılınca otomatik başlar). Delivery gerekmedi.
- **Bugün:** `<body onresize=print()>` → `onresize` **kendiliğinden ateşlenmez.** Bir şeyin
  **boyutu değişmeli.** Bu yüzden event'i tetikleyecek bir **delivery mekanizması** gerekti.

## Çözüm adımları

### Aşama 1 — İzinli tag (kapsamlı liste)
`<script>` → bloklu. Geniş tag listesiyle Python taraması → izinli: **`body`** (ve işe yaramayan `desc`).

### Aşama 2 — İzinli event (kapsamlı liste)
`<body EVENT=1>` taraması → izinli: **`onresize`** (+ onabort, onstorage — tetiklemesi zor).
Kullanışlı olan `onresize`: element/pencere boyutu değişince ateşlenir.

### Aşama 3 — Delivery (yeni beceri: iframe + resize)

Problem: `onresize` normal açılışta ateşlenmez. Çözüm: zafiyetli sayfayı bir **iframe** içine koy,
iframe yüklenince **boyutunu değiştir** → içerik resize olur → iç `<body onresize>` ateşlenir.

Exploit (exploit server Body):
```html
<iframe src="https://LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E"
        onload="this.style.width='100px'"></iframe>
```

- `src` = zafiyetli sayfa + payload. `"><body onresize=print()>` → URL-encoded:
  `%22%3E%3Cbody%20onresize=print()%3E` (`%22`=`"`, `%3C`=`<`, `%3E`=`>`, `%20`=boşluk).
- `onload="this.style.width='100px'"` = iframe yüklenince genişliğini değiştir → **resize tetikler.**
- **iframe = kap, boyut değişimi = tetikleyici.** iframe tek başına yetmez; asıl olay resize.

**Deliver to victim** → kurbanın tarayıcısında iframe yüklenir, resize olur, `print()` çalışır → çözüldü.
(Hedef `alert` değil **`print()`** — lab bunu istiyor.)

## Remediation (defender)

1. **Allowlist filtresine güvenme — tam context-aware output encoding yap.** Bu lab kanıtlıyor:
   "çoğu tag bloklu" bile yetmedi; `body` + `onresize` sızdı. Tag/attribute kara/beyaz listesi
   her zaman eksik kalır.
2. **CSP:** inline event handler'ları engelle (`unsafe-inline` kapalı, `script-src` kısıtlı) →
   `onresize=...` çalışmaz.
3. **Framework'ün güvenli template engine'i** (otomatik escape) + olgun sanitizer (DOMPurify).

## Zincir bağı

- XSS #8: filter bypass, otomatik ateşlenen event (SVG onbegin).
- XSS #9 (bu): daha sıkı filter + **dış tetik gerektiren event** → iframe/resize delivery.
- Ortak beceri: **enumerate what's allowed** — ve bu sefer "**kapsamlı** enumerate et" dersi.
- Gerçek hedef köprüsü: Smartsheet recon'da da wordlist kalitesi = görüş alanın. Zayıf liste = kör nokta.

## Köşe taşı

> Enumeration'ın görüşü wordlist'in kadardır — eksik liste false negative üretir. Ve bir event
> kendiliğinden ateşlenmiyorsa, onu **tetikleyecek** bir mekanizma (iframe + resize gibi) kurman gerekir.
