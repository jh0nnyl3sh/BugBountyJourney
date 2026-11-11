# Information Disclosure #2 — Source code disclosure via backup files

- **Seviye:** Apprentice
- **Sınıf:** Information Disclosure
- **Yöntem:** Körleme (hızlı solve — exploit server / Burp gerekmedi)
- **Araç:** Sadece tarayıcı

---

## Zafiyet nedir?

Geliştirici, kaynak kodun bir **yedeğini** (`.bak`) sunucuda erişilebilir bırakmış. Bu dosya
tarayıcıda **düz metin** olarak açılır (çünkü `.bak` uzantısını sunucu "çalıştırılacak kod" diye
tanımıyor, dosyanın içeriğini olduğu gibi döküyor). İçinde **hardcoded (sabit kodlanmış) bir sır**
vardı → veritabanı parolası.

## Çözüm adımları

1. **`/robots.txt`** → gizli dizini ele verdi: `/backup`.
2. **`/backup`** → içinde `ProductTemplate.java.bak` dosyası.
3. Dosyayı aç → kaynak kod düz metin olarak göründü.
4. İçinde DB bağlantısı:
   ```java
   ConnectionBuilder.from(
       "org.postgresql.Driver", "postgresql", "localhost", 5432,
       "postgres",  // veritabanı
       "postgres",  // kullanıcı
       "pyg411z56w117iji0hsxbnwn1hyhn1b6"  // <-- ŞİFRE (son string)
   )
   ```
5. Son string = **hardcoded parola** → lab'a gönderildi → çözüldü.

## Sızan kritik bilgi

- **Hardcoded DB parolası** (PostgreSQL, `localhost:5432`).
- Argüman sırası: driver, tip, host, port, db, kullanıcı, **parola** (sonuncu).

## Bonus gözlem (refleks gelişiyor)

Aynı kodda:
```java
String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
```
Kullanıcı girdisi (`id`) doğrudan query'ye gömülmüş → bu **SQL injection**. Bu lab onu istemiyor
ama kaynak kodu okurken injection noktalarını otomatik görmek, gerçek sahada çok değerli bir alışkanlık.

## Neden çalıştı? (kök neden)

1. Yedek dosya **web root içinde** erişilebilir bırakıldı (`/backup/...`).
2. `/robots.txt` gizli dizini **ifşa etti** (robots.txt gizleme değil, tersine haritadır).
3. Sırlar **koda sabit kodlanmış** (hardcoded) — kaynağa erişen herkes alır.

## Remediation (defender tarafı)

1. **Yedek/geçici dosyaları web root dışında tut:** `.bak`, `.old`, `~`, `.swp` asla sunucudan
   servis edilmemeli. Deploy sürecinde bunları hariç tut.
2. **Sırları koddan çıkar:** Parola/anahtarlar environment variable veya secret manager'da;
   asla kaynak kodda hardcoded değil.
3. **`robots.txt`'i sır saklamak için kullanma:** Orada yazdığın her dizin saldırgana ipucudur.
   Hassas yolları erişim kontrolüyle koru, gizlilikle değil.
4. **Dizin listelemeyi kapat** ve bilinmeyen uzantıların düz metin servis edilmesini engelle.

## Zincir bağı

- **Info Disclosure #1 (error messages):** hata mesajından **sürüm** sızdı.
- **Info Disclosure #2 (bu):** yedek dosyadan **kaynak kod + parola** sızdı.
- Ortak ilke: uygulama, görünmemesi gereken şeyi **erişilebilir** bırakıyor. #1'de tetikleyerek,
  #2'de tahmin edilebilir yol (`/robots.txt` → `/backup`) ile ulaştık. İkisi de **recon**'un kalbi.

## Köşe taşı

> Sunucuda duran her dosya potansiyel bir sızıntıdır. `robots.txt` gizleme aracı değil, harita;
> `.bak` dosyaları kaynak kodu ifşa eder; hardcoded sırlar koda erişen herkesindir.
