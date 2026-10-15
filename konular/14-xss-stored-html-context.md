# 14. XSS Lab #2: Stored XSS — HTML context, nothing encoded — 15.10.2026

> Reflected'ın tehlikeli kardeşi. Aynı payload, bambaşka etki: kalıcı ve herkese çalışıyor.

**Lab:** PortSwigger → Cross-site scripting → Stored → *Stored XSS into HTML context with nothing encoded* (Apprentice)
**Görev:** `alert` çalıştıran stored XSS gerçekleştir.

## Çözüm
1. Lab bir blog. Bir yazıya girilip **comment (yorum)** bölümü bulundu.
2. Yorum kutusuna `<script>alert(1)</script>` yazıldı, gönderildi.
3. Yorum sunucuda **saklandığı** için, blog yazısı her açıldığında kod çalıştı → **Solved.** ✅

## Reflected vs Stored (kritik ayrım)
| | Reflected | Stored |
|---|-----------|--------|
| Kod nerede | İstekte gelir, aynı yanıtta yansır | Sunucuda **saklanır** (yorum/profil/mesaj) |
| Kime çalışır | Sadece o linke tıklayan kurban | Sayfayı açan **herkes** |
| Kalıcı mı | Hayır, tek seferlik | **Evet**, bir kere ekle, bekler |
| Teslimat | Kurbanı özel linke tıklatman gerekir | Otomatik; tuzak kurulur |

## Altın ders — etki (impact) önemlidir
- **Teknik aynı** (`<script>alert(1)</script>`), ama değer tekniğin havalılığından değil **etkiden** gelir.
- Reflected genelde orta severity; Stored çok daha yüksek severity + daha yüksek ödül (herkesi etkiler).
- Rapor yazarken anlatılan şey etkidir: "alert açtım" değil, "tüm kullanıcıların oturumunu çalabilecek kalıcı XSS".

## Kök sebep (ikisinde de aynı)
- Site kullanıcı girdisini encode/sanitize etmeden HTML'e basıyor.
- Farkı yaratan: girdinin *nerede saklandığı* (anlık yanıt mı, veritabanı mı) → etkiyi belirler.

## Zincir bağı
- "Tahmin edilemez ID" lab'ında carlos'un ID'si yorumlarda sızıyordu. Yorumlar = kullanıcının yazdığının başkalarına gösterildiği yer = Stored XSS'in en sevdiği av alanı.
