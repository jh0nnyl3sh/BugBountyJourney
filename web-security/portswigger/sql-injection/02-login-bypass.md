# SQLi Lab #2: Authentication bypass (login bypass) — 23.10.2026

> Dünkü `' OR 1=1 --` mantığının login formunda tekrarı — ama yeni bir incelik: iki alan (username + password) ve operator precedence. "Balyoz" ile "neşter" farkını öğreten lab.

**Lab:** PortSwigger → SQL injection → *SQL injection vulnerability allowing login bypass* (Apprentice)
**Görev:** `administrator` kullanıcısı olarak giriş yap (şifreyi bilmeden).

## Enjeksiyon noktası
- "My account" → login formu. İki alan: username + password.
- Tahmin edilen arka sorgu:
```sql
SELECT * FROM users WHERE username = 'GIRDI_U' AND password = 'GIRDI_P'
```
Bir satır dönerse giriş başarılı sayılıyor.

## Çözüm A — "neşter" (hedefli, doğru yöntem)
```
username:  administrator'--
password:  (boş / herhangi bir şey, örn. 1)
```
Sorgu:
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = '1'
```
- `'` → username tırnağını kapattı (breakout).
- `--` → arkadaki ` AND password = '...'` kontrolünü komple yorumladı → şifre hiç sorulmadı.
- Geriye tam olarak `WHERE username = 'administrator'` kaldı → **sadece ve kesin administrator** döner → **Solved.** ✅
- Avantaj: kimi istediğini SEN seçtin; satır sırasına bağlı değil, garanti.

## Çözüm B — "balyoz" (çalışır ama hedefsiz)
```
username:  administrator
password:  ' OR 1=1 --
```
Sorgu:
```sql
SELECT * FROM users WHERE username = 'administrator' AND password = '' OR 1=1 --'
```
**Operator precedence (kritik):** SQL'de `AND`, `OR`'dan önce bağlanır. Cümle şöyle okunur:
```sql
WHERE ( username = 'administrator' AND password = '' ) OR ( 1=1 )
```
- `1=1` her zaman doğru → `OR` yüzünden tüm koşul doğru olur → sorgu **TÜM kullanıcıları** döndürür.
- `username = 'administrator'` kısmı burada işe yaramadı; `OR 1=1` koşulu komple havaya uçurdu. Yani "administratorüm" demedim, "herkesi getir" dedim.
- Uygulama dönen listeden genelde **ilk satırı** alıp giriş yapar. Bu lab'da ilk satır administrator olduğu için çözüldü — yani **satır sırası sayesinde**, hedefleme sayesinde değil. Başka tabloda ilk satır `wiener` olsaydı lab çözülmezdi.

## Ders (ezber değil, mantık)
- `OR 1=1` = koşulu komple aç, herkesi getir → **kaba balyoz**, hangi kullanıcı olacağın tabloya kalır.
- `administrator'--` = kullanıcıyı adıyla hedefle, kontrolü kes → **neşter**, kimi istediğini sen seçersin.
- `--`'ın asıl gücü: "beni engelleyen arka kısmı (burada password kontrolü) yok et."
- Hangi ALANA yazdığın önemli: sorgunun o alanı nasıl kullandığına bağlı. Precedence'ı hesaba kat.

## 🛡️ Nasıl kapatılır (remediation)
Kök sebep yine aynı: girdi, SQL metnine string birleştirmeyle karışıyor (veri = kod).

- **Asıl çözüm — parameterized queries:** username ve password ayrı parametre olarak gitsin.
  ```csharp
  var sql = "SELECT * FROM users WHERE username = @u AND password = @p";
  cmd.Parameters.AddWithValue("@u", username);
  cmd.Parameters.AddWithValue("@p", password);
  ```
  Böylece `administrator'--` yazsan bile bu "tam olarak böyle bir username" diye aranır, kod olmaz.
- **Ek katmanlar (defense-in-depth):**
  - Şifreler **plaintext karşılaştırılmaz**; hash'lenir (bcrypt/Argon2) → `password = '...'` diye bir string karşılaştırması zaten olmamalı. Bu, login SQLi'sini doğal olarak zorlaştırır.
  - Hesap kilitleme / rate limiting (brute-force'a karşı), başarısız denemeleri logla.
- **İlke:** kimlik doğrulama kararı asla string birleştirilmiş bir sorgunun "bir satır döndü mü" sonucuna bırakılmaz.

## Zincir bağı
- Dün (#1): tek alan (`category`), `' OR 1=1 --` ile filtreyi genişlettik.
- Bugün (#2): iki alan + precedence. `--` ile password kontrolünü kesmeyi ve belirli bir kullanıcıyı hedeflemeyi öğrendik.
- Sıradaki (#3 hedef): UNION ile başka tablolardan veri çekmek — "koşulu değiştirmek"ten "veri sızdırmak"a geçiş.
