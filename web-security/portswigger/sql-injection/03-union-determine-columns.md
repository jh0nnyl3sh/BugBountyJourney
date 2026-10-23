# SQLi Lab #3: UNION attack — sütun sayısını belirleme — 23.10.2026

> İlk UNION lab'ı ve ilk Practitioner seviyesi. "Koşulu değiştirmek"ten "başka tablodan veri sızdırmak"a geçişin ilk adımı: her UNION saldırısının zorunlu ön hazırlığı.

**Lab:** PortSwigger → SQL injection → UNION attacks → *Determining the number of columns returned by the query* (Practitioner)
**Görev:** Orijinal sorgunun kaç sütun döndürdüğünü, NULL'larla başarılı bir UNION SELECT yaparak belirle.

## UNION nedir (kısa)
`UNION`, ikinci bir SELECT'in sonucunu ilkinin çıktısına **alt alta zımbalar**. İkinci SELECT başka bir tablodan veri çekebilir ve sonuç, ilkinin göründüğü yerde sayfaya basılır → saldırgan buradan veri sızdırır.

## UNION'ın iki demir kuralı
1. İki SELECT de **aynı sayıda sütun** döndürmeli.
2. Eşleşen sütunların **veri tipleri uyumlu** olmalı.
Bu yüzden hiçbir UNION'a doğrudan "şifreleri çek" diye başlanmaz; önce (a) sütun sayısı, (b) hangi sütun metin tutuyor çözülür. Bu lab = (a) adımı.

## Çözüm
Enjeksiyon noktası: `?category=` filtresi.

**Adım 1 — sayıyı bul (ORDER BY):**
```
' ORDER BY 1--   → OK
' ORDER BY 2--   → OK
' ORDER BY 3--   → OK
' ORDER BY 4--   → Internal Server Error
```
`ORDER BY 4` hata, `3` çalışıyor → sorgu **3 sütun** döndürüyor. (İlk hata veren sayının bir eksiği cevap.)

**Adım 2 — kanıtla (UNION SELECT NULL):**
```
' UNION SELECT NULL,NULL,NULL--
```
Hata kayboldu, sorgu başarılı döndü (fazladan boş satır) → **Solved.** ✅

## Neden böyle (mantık)
- **3 adet NULL** → Kural 1 (aynı sütun sayısı) ORDER BY ile bulunan 3'le eşleşti.
- **NULL seçilmesi** → Kural 2'yi (tip uyumu) by-pass eder; NULL her tiple uyumlu, böylece "hangi sütun metin/sayı" derdine girmeden yalnızca sayıyı test edersin.
- **ORDER BY vs UNION SELECT:** ORDER BY sayıyı *söyler* ama DB'ye satır ekletmez; UNION SELECT sayıyı *kanıtlar* (gerçek fazladan satır). Lab'ın çözüm şartı ikincisiydi.
- "Hata da bilgidir": doğru sütun sayısını, tam olarak hatanın başladığı/bittiği yeri okuyarak buldun.

## 🛡️ Nasıl kapatılır (remediation)
Teknik adımı farklı olsa da kök sebep ve çözüm aynı aileden: girdi SQL metnine karışıyor → **parameterized queries.** Parametreli sorguda `' UNION SELECT ...` sadece garip bir kategori metni olarak aranır, SQL yapısına dokunamaz. Ek olarak: DB hata mesajlarını kullanıcıya ham göstermeme (generic error) saldırganın geri bildirimini azaltır; least privilege ile UNION'la ulaşılabilecek tabloları sınırla.

## Zincir bağı
- #1/#2: koşulu değiştiriyorduk (OR 1=1, --). #3: veri sızdırmanın altyapısını kuruyoruz.
- Sıradaki adımlar: (2) hangi sütun metin tutuyor → bir NULL'u `'abc'` ile değiştirip hata vermeyen sütunu bulmak; (3) o sütuna gerçek username/password çekmek.
