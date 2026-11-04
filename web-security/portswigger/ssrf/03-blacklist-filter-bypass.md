# SSRF Lab #3: blacklist (filtre) atlatma — 04.11.2026

> SSRF'in bypass ayağı. Aynı hedef (localhost admin), ama bir kara liste bariz girdileri engelliyor. Çözüm: aynı yeri filtrenin tanımadığı bir biçimde yazmak. Dersin özü: kara listeler kırılgandır → savunma allowlist olmalı.

**Lab:** PortSwigger → SSRF → *SSRF with blacklist-based input filter* (Practitioner)
**Görev:** Filtreyi atlatıp localhost admin paneline ulaş, carlos'u sil.

## Filtre neyi engelliyordu
- `http://localhost/admin` ve `http://127.0.0.1/admin` → **400 Bad Request** (blacklist bunları yakalıyor).
- Ayrıca `admin` kelimesi de engelli.

## Bypass — iki kol
**1. IP'nin alternatif gösterimi** (aynı loopback, farklı metin):
- `[::1]` denendi → filtreyi **geçti ama 500** (IPv6 loopback; admin IPv4 `127.0.0.1`'de, orada kimse yok → ulaşamadı).
- `127.1` → filtreyi geçti **ve** IPv4 loopback'e denk geldi → ulaştı. ✅
  (Diğer seçenekler: ondalık `2130706433`, octal `0177.0.0.1`.)

**2. Engellenen kelimeyi gizle:** `admin` engelli → **`/Admin`** (büyük A). Sunucu path'i case-insensitive çözüp yine admin'e gider, filtre düz `admin` string'ini görmez.

## Çözüm
```
stockApi=http://127.1/Admin            -> 200 + admin paneli
stockApi=http://127.1/Admin/delete?username=carlos   -> carlos silindi
```
→ **Solved.** ✅

## Dersler
- **Geçmek ≠ ulaşmak:** `[::1]` filtreyi geçti ama hedefe varmadı (500). Bir payload'ın filtreyi aşması, işe yaradığı anlamına gelmez; gerçekten doğru hedefe ulaştığını (200 + beklenen içerik) doğrula.
- **Kara listeler kırılgandır:** tüm kötü varyasyonları önceden sayıp engelleyemezsin; hep dışarıda kalan bir gösterim olur (`127.1`, decimal, octal, IPv6, case, encoding...). Aynı hedefin sonsuz yazımı vardır.

## 🛡️ Nasıl kapatılır (remediation)
- **Allowlist > blacklist (altın kural):** "kötüleri engelle" değil, **"sadece izinli host/şema/porta izin ver, gerisini reddet".** İzinliler sonlu ve bilinir; kötüler sonsuz. Bu kural tüm input validation'da geçerli.
- **Girdiyi normalize et + çözümlenen IP'yi kontrol et:** URL'yi parse edip host'u IP'ye çöz, çözülen IP iç/loopback aralığındaysa reddet (127.1/decimal/octal/IPv6 hepsi aynı IP'ye çözülür → tek noktada yakalanır). Redirect takip etme.
- Path için de case/encoding normalize edip yetki kontrolünü server-side yap; admin arayüzü iç ağda bile auth istesin.

## Zincir bağı
- Lab #1 localhost, #2 iç ağ keşfi, #3 filtre atlatma. "Aynı hedef, farklı kılık" mantığı = dünkü XSS `javascript:` bypass'ının kardeşi; ileride WAF atlatmanın temeli.
