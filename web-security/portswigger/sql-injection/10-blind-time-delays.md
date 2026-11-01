# SQLi Lab #10: Blind SQLi — time delays (PostgreSQL) — 01.11.2026

> Blind'ın üçüncü sinyali: ZAMAN. Uygulama ne içerik ne hata farkı gösterdiğinde, DB'yi bekleterek evet/hayır alırsın. Bu lab sadece sinyali kanıtlıyor (veri çıkarma yok).

**Lab:** PortSwigger → SQL injection → Blind → *Blind SQL injection with time delays* (Practitioner, PostgreSQL)
**Görev:** Yanıtı ~10 sn geciktiren bir time-based injection tetikle.

## Omurga aynı, sinyal değişti
Blind'ın değişmezi: evet/hayır oracle + karakter karakter çıkarma. Değişen tek şey sinyal:
- responses → içerik (Welcome back)
- errors → HTTP kodu (500/200)
- **time delays → süre (yavaş/hızlı)**
Kapalı kutu hep cevap verir; bu sefer "evet"i **bekleterek** söylüyor.

## Çözüm
PostgreSQL → gecikme fonksiyonu `pg_sleep(saniye)`, string birleştirme `||`.
```
TrackingId=xxxxx' || pg_sleep(10)--
```
- `'` → string'den çık (breakout).
- `|| pg_sleep(10)` → sorguya 10 sn'lik bekleme ekle.
- `--` → kalanı kes, cümle geçerli kalsın.
Yanıt ~10 sn gecikmeli geldi → **Solved.** ✅ (Bu lab'da koşula gerek yok; sadece gecikmeyi kanıtlıyoruz.)

## Ne zaman time-based? (escalation)
1. Veri doğrudan görünüyor mu (in-band/UNION)? → en kolay.
2. İçerik farkı var mı (conditional responses)?
3. Hata farkı var mı (conditional errors)?
4. Hiçbiri yoksa → **time-based.** En yavaş/gürültülü, son çare. Uygulama gözlemlenebilir hiçbir şey sızdırmadığında.

## 🛡️ Nasıl kapatılır (remediation)
- **Parameterized queries** — tek gerçek çözüm; girdi SQL yapısına karışamazsa `pg_sleep` de enjekte edilemez.
- Önemli: time-based, **çıktının/hatanın tamamen gizli olduğu** durumda bile çalışır → "çıktıyı/hatayı gizlersem güvenli olurum" yanılgısını kesin çürütür. Gizlemek zafiyeti yok etmez.
- Least privilege; sorgu zaman aşımı/anormal gecikme monitoringi (tespit için, önlem değil).

## Zincir bağı
- Blind #1 (responses) → #2 (errors) → #3 (time): aynı iskelet, sinyal gittikçe sinsileşti.
- Sıradaki (info-retrieval): bu zaman sinyalini bir `CASE WHEN ... THEN pg_sleep(10)` ile **koşula bağlayıp** gerçek şifreyi çıkaracağız (conditional errors'daki CASE mantığının zaman versiyonu).
