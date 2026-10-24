# SQLi Lab #5: UNION attack — başka tablodan veri çekme — 24.10.2026

> UNION serisinin finali. Önceki üç adım (sütun sayısı, metin sütunu) buraya hizmetti: artık gerçek veriyi (username/password) sızdırıp o veriyle giriş yapıyoruz.

**Lab:** PortSwigger → SQL injection → UNION attacks → *Retrieving data from other tables* (Practitioner)
**Görev:** `users` tablosundan administrator'ın bilgilerini çek ve administrator olarak giriş yap.
**Lab'ın verdiği isimler:** tablo `users`, sütunlar `username` ve `password`.

## Çözüm (uçtan uca)
Enjeksiyon noktası: `?category=` filtresi (bir liste döndürüyor).

1. **Sütun sayısı:** `' ORDER BY 1--` ... → `' UNION SELECT NULL,NULL--` çalıştı, tek NULL hata verdi → **2 sütun**.
2. **Metin sütunu:** `' UNION SELECT 'test',NULL--` ve `' UNION SELECT 'test','test2'--` çalıştı → **her iki sütun da string**.
3. **Gerçek veriyi çek:**
   ```
   ' UNION SELECT username, password FROM users--
   ```
   Tablodaki tüm kullanıcı/şifre çiftleri ekrana döküldü.
4. Çıktıdan **administrator** satırının şifresi alındı → "My account" login'de `administrator` + o şifre ile giriş → **Solved.** ✅

## Kritik kavram: düz metin vs gerçek veri
- `' UNION SELECT 'administrator','password'--` → tırnak içindekiler **senin uydurduğun sabit string'ler**; ekrana aynen basılır, veritabanından gelmez.
- `' UNION SELECT username, password FROM users--` → `username`/`password` **tırnaksız = sütun adları**; `FROM users` = "bu değerleri `users` tablosundan getir". İşte gerçek veri buradan gelir.
- **Terim netliği (önemli):** `users` bir TABLO; `username`/`password` o tablonun SÜTUNLARI. Büyük saldırılarda önce tabloları, sonra sütunları enumerate edersin — ikisini karıştırmak sorguyu kırar.

## Neden dört UNION lab'ı bu sırayla?
Her adım, bir sonraki için zorunlu bir bilinmeyeni çözer:
- Kaç sütun? (bilmezsen UNION geçersiz, Kural 1)
- Hangi sütun metin? (bilmezsen çaldığın string'i nereye koyacağını bilemezsin, Kural 2)
- Ancak bu ikisi çözülünce gerçek veri çekilebilir.
Mantıksal zorunluluk; ezber değil.

## 🛡️ Nasıl kapatılır (remediation)
- **Asıl çözüm — parameterized queries:** girdi SQL yapısına karışamaz; `' UNION SELECT ...` sadece garip bir kategori metni olur.
- **Şifreleri plaintext saklama — hash'le (bcrypt/Argon2):** Bu lab'da şifreler düz metin olduğu için çekilince anında kullanılabildi. Hash'li olsalardı, tablo sızsa bile saldırgan düz şifreyi alamazdı. (SQLi'yi önlemez ama etkisini ciddi azaltır — defense-in-depth.)
- **Least privilege:** uygulamanın DB kullanıcısı yalnızca gereken tablolara erişsin; UNION'la `users` gibi hassas tablolara ulaşım sınırlansın.
- **Generic error / hassas tabloları ayırma:** ham DB hataları saldırgana sütun tipi/yapı sızdırır.

## Zincir bağı
- #3 (sütun sayısı) + #4 (metin sütunu) → #5'te birleşti: gerçek veri sızdırma + o veriyle auth bypass (#2'deki login hedefinin UNION'la farklı yoldan tekrarı).
- Sıradaki: veritabanının kendisini keşfetmek (DB türü/sürümü, hangi tablolar/sütunlar var) — isim verilmediğinde kazarak bulmak.
