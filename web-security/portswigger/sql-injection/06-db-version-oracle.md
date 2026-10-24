# SQLi Lab #6: Examining the DB — tür/sürüm belirleme (Oracle) — 24.10.2026

> "Examining the database" serisinin ilki. Artık sadece veri çekmiyoruz; önce hedefi tanıyoruz — çünkü DBMS'e göre syntax değişir, doğru silahı seçmek için türü/sürümü bilmek gerekir.

**Lab:** PortSwigger → SQL injection → Examining the database → *Querying the database type and version on Oracle* (Practitioner)
**Görev:** Veritabanının sürüm bilgisini UNION ile ekrana döktür. (Lab DBMS'in Oracle olduğunu veriyor.)

## Çözüm
Enjeksiyon noktası: `?category=` filtresi.

1. **Sütun sayısı — Oracle quirk ile:** `' UNION SELECT NULL--` → **hata.** Çünkü Oracle'da bir `SELECT` tablosuz çalışmaz; tablosuz seçim için sahte tablo **`dual`** gerekir. Doğrusu:
   ```
   ' UNION SELECT NULL FROM dual--
   ' UNION SELECT NULL,NULL FROM dual--   → çalıştı
   ```
   → **2 sütun** (ve ikisi de metin alıyor).
2. **Sürümü çek:** Oracle'da sürüm bilgisi `v$version` tablosunun `BANNER` sütununda tutulur (google ile bulundu). `dual`'a gerek yok, `v$version` gerçek tablo:
   ```
   ' UNION SELECT BANNER, NULL FROM v$version--
   ```
   Sürüm satırları ekrana döküldü → **Solved.** ✅

## Kritik kavram: DBMS'e özgü syntax (neden fingerprinting?)
- Her DBMS farklı syntax/işlevle çalışır: tablosuz select (Oracle `dual`), sürüm sorgusu (Oracle `v$version.BANNER`, MySQL/MSSQL `@@version`, PostgreSQL `version()`), string birleştirme, yorum karakterleri hepsi değişir.
- O yüzden saldırının erken adımı **DBMS'i ve sürümü tespit etmektir** — ondan sonraki her payload'ı doğru dialect'te yazarsın. Yanlış DBMS varsayımı = sürekli syntax hatası.
- Bu lab türü lab'da veriyor; gerçek dünyada hata mesajları, davranış farkları ve sürüm sorgularıyla **sen tespit edersin.**

## Zincir bağı
- UNION serisinde (sütun sayısı → metin sütunu → veri çekme) öğrendiğin iskeleti burada aynen kullandık; tek yenilik Oracle'ın `dual` ve `v$version` incelikleri.
- Oracle quirk (`FROM dual`), "önce hedefi tanı, sonra dialect'e göre yaz" ilkesinin ilk somut örneği.

## 🛡️ Nasıl kapatılır (remediation)
- Asıl kök sebep ve çözüm değişmedi: **parameterized queries** — girdi SQL yapısına karışamazsa ne sürüm çekilir ne başka şey.
- **Hata mesajlarını kullanıcıya ham gösterme (generic error):** Ham DBMS hataları, türü/sürümü ve yapıyı saldırgana bedava verir (fingerprinting'i kolaylaştırır).
- **Not:** "Sürüm açığa çıktı" tek başına düşük önemde; asıl mesele sürümü çektirebilen **SQLi'nin kendisi.** Remediation SQLi'ye odaklanır; sürüm gizleme sadece defense-in-depth.
- Least privilege: uygulama DB kullanıcısının `v$version` gibi sistem view'larına gereksiz erişimi kısılabilir.
