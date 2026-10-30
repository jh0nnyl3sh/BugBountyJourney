# Öz-değerlendirme — Gün 25 (30.10.2026)

> Milestone notu. 50. günde buna geri dönüp kıyasla.

## Nerede
- 25 gün, **23 lab**: Access Control 7, XSS 7, SQL Injection 9.
- **Apprentice → Practitioner** geçişi yapıldı.
- UNION tam zinciri + examining DB + **blind SQLi 2 tür** çözüldü.
- **İlk otomasyon** yazıldı (Python blind SQLi script).
- Her lab için write-up + remediation; scoreboard'lu repo.

## Güçlü yanlar
- **Metacognition:** kendi öğrenmeni yönetiyorsun (sarmal tekrarı sen önerdin, tempoyu sen sorguladın, "gerçek hayata uygun mu / CEH değer mi" diye sen sordun).
- Körleme disiplini, zorlanmayı sevme, feedback'i alıp **uygulama**, her şeyi belgeleme.

## Zayıf yanlar
1. **Precision (1 numaralı açık):** tablo↔sütun, `USER`↔`username`, `SELECT *`↔`dual`, off-by-one (uzunluk), `SUBSTRING`↔`SUBSTR`, eksik charset, görevin tam metnini atlama. Gerçek sömürüde tek karakter hata = eksik/başarısız.
2. Yeni durumu **eski refleksle** çözmeye çalışma (uyup uymadığını kontrol etmeden).
3. **Articulation ince:** recount'ta mekanizmayı ("neden") atlayıp adımı söyleme eğilimi.
4. **Darlık:** henüz XSS + SQLi + access control; SSRF/auth/CSRF/recon/gerçek hedef yok.

## Nasıl daha iyi (aksiyon)
- **Her ders başı precision kontrol listesi** (kontrollü körleme): doğru tablo/sütun adı · bu DBMS'in doğru fonksiyonu/syntax'ı · tırnak tipi + off-by-one · görevin tam metni.
- **"Ateş etmeden önce sınıflandır":** payload öncesi source/sink/context-tipi/teknik cümlesi.
- **Mekanizmayı yaz:** her lab sonrası "neden çalıştı"yı tek cümle kendi kelimelerinle.
- Altındaki fundamentals'ı (HTML/JS parsing, encoding, SQL semantiği) düzenli tazele.

## Mindset (kendi sözlerinden)
- Gerçek hedefe sabırsız ama "şu an yetersizim, kendimi kandırmam, çalışınca geleceğim" diyebiliyor — olgun ve sağlıklı.
- "Siber güvenlik sevdiğim bir hobi; insan sevdiği işi güzel yapar." → en güçlü motivasyon kaynağı bu.

## Genel kanaat
Ciddi, farkında, doğru alışkanlıklarla ilerleyen bir öğrenci. Henüz erken (25. gün) ve gerçekten test edilmedi; ama temel ve alışkanlıklar doğru olanlar. Başarıyı en çok belirleyecek iki şey: (1) precision zaafını güce çevirmek, (2) zamanı gelince lab'lardan gerçek hedeflere geçmek.
