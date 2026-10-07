# Bug Bounty Öğrenme Günlüğü

> Başlangıç: 06.10.2026 • Bir yıllık kontrol noktası: 06.10.2027
> Her yeni konuya geçmeden önce bu dosyayı bir oku, tekrar et, sonra devam et.

---

## Yol Haritası (Kuşbakışı)

1. **Temeller** — İnternet/web nasıl çalışıyor (HTTP, DNS, oturum)
2. **Araçlar** — Burp Suite başta olmak üzere
3. **Zafiyet sınıfları** — OWASP Top 10 ve ötesi (IDOR, XSS, SQLi, SSRF...)
4. **Pratik** — PortSwigger Academy, TryHackMe, HackTheBox (yasal lab'lar)
5. **Gerçek sahne** — HackerOne, Bugcrowd, Intigriti

**Asıl merdivenim (bilgi basamakları):**
Ağ + Linux temelleri → Web uygulama güvenliği (OWASP + Burp) → Bir alanda uzmanlaşma
Sertifika hedefi (şimdilik tek): **BSCP** (Burp Suite Certified Practitioner)

**Altın kural:** Hedef = bug bounty (motivasyon). Günlük iş = basamakları sırayla, sindirerek geçmek. Para, bilginin doğal sonucudur.

---

## ✅ İşlenen Konular

### 1. Web Nasıl Çalışır? — İstek/Yanıt Döngüsü
Tarayıcıya adres yazıp Enter'a basınca:
1. **DNS** — alan adını IP'ye çevirir (internetin telefon rehberi)
2. **Bağlantı** — TCP three-way handshake (3'lü el sıkışma); HTTPS üstüne şifreli zarf ekler
3. **HTTP İsteği (Request)** — tarayıcı "bana şunu gönder" der → *avcılığın kalbi burada*
4. **HTTP Yanıtı (Response)** — sunucu işler, veritabanına bakar, HTML + durum kodu döner
5. **Çizim** — tarayıcı HTML/CSS/JS'i alıp sayfayı ekrana çizer

**Özü:** Bütün web şu döngüde döner → *İstek git, yanıt gel.*

### 2. HTTP İsteğinin İçi
Düz metin bir istek şöyle görünür:
```
GET /profil HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
```
- **İlk satır:** metot (GET) + yol (/profil) + versiyon
- **Host:** hangi siteyle konuşuyorum
- **User-Agent:** kendimi nasıl tanıtıyorum (yalan söyleyebilir!)
- **Cookie:** kimliğim (session ID burada taşınır)

**Metotlar:** GET (getir), POST (gönder), PUT (güncelle), DELETE (sil)
**Avcı sorusu:** "Bu sayfa GET bekliyor, ben DELETE gönderirsem ne olur?"

### 3. Oturum (Session) Mantığı — "Sunucu beni nasıl tanıyor?"
- **HTTP durumsuzdur (stateless):** sunucunun hafızası yok, her istekte seni ilk kez görür.
- **Çözüm — bileklik benzetmesi:** Giriş yapınca (POST) sunucu sana rastgele, tahmin edilemez bir **session ID** üretir.
  - Sunucu → sana verirken: `Set-Cookie` (geliş yönü)
  - Sen → sunucuya gösterirken: `Cookie` (gidiş yönü)
  - Sunucu kendi tarafında da "bu ID = Uğur" kaydını tutar.
- Böylece durumsuz protokolün üstüne çerezle sahte bir "hafıza" kurulur.

**Oturum zafiyetlerinin doğduğu 4 soru (cevap "hayır" ise zafiyet var):**
1. Session ID gerçekten tahmin edilemez mi? (yoksa session prediction)
2. Çerez çalınabilir mi? (session hijacking)
3. Çıkışta (logout) gerçekten iptal oluyor mu?
4. Koruma etiketleri var mı? (HttpOnly, Secure, SameSite — ileride işlenecek)

> Not: Tehlike sadece "ben kaptırırsam" değil; bazen garson bilekliği en baştan kötü tasarlamıştır.

### 4. İlk Zafiyet: IDOR (Insecure Direct Object Reference)
**Mantık:** `example.com/profil?id=1004` → `id=1005` yazınca başkasının verisi açılıyorsa IDOR var.

**Kök sebep:** Yetki kontrolünün (authorization) yapılmaması.
- **Authentication:** "Sen kimsin?" → giriş yaptın mı? (kimlik)
- **Authorization:** "Bunu görmeye hakkın var mı?" (yetki)
- IDOR = kimliğin gerçek, ama erişimin kaçak. Garson seni içeri alıyor ama mutfağa dalınca durduran yok.

**Sinsi kardeşi — "tahmin edilemez" sanılan ID'ler:**
- `id=8f14e45fceea167a5a36dedd4bea2543` gibi karmaşık ID görünce "burada iş yok" DEME.
- **Tahmin edilemezlik ≠ yetkilendirme.** Kapı kilitli değil, sadece numarası okunması zor yazılmış.
- O karmaşık değer aslında sıralı bir ID'nin hash'i olabilir, bir yerde sızıyor olabilir, ya da yetki hiç kontrol edilmiyordur.
- Asıl ödüller, çoğunluğun "karmaşık, geçeyim" dediği yerde saklı. **Sen geçme, sor.**

**Yanıtı okuma inceliği:** 403 (bilerek durduruyor, iyi) / 404 (yok gibi davranıyor, bazen bilgi sızdırır) / 200 ama boş → yanıtın *ne* olduğu kadar *nasıl* olduğu da ipucudur.

**Akraba teknik:** `/admin`, `/users` gibi denemeler IDOR değil → **içerik/dizin keşfi** (content discovery). Ayrı kas, ileride işlenecek.

---

## ⚖️ Etik Çizgi (Her zaman cepte)
- İzinsiz siteye test = **suç** (VPN'li de olsa, yavaş da atsan, niyetin iyi de olsa). TCK kapsamında.
- İki altın şart: **Scope** (program neyi test edebileceğini yazıyla söyler, dışına çıkma) + **Yetki** (yazılı izin yoksa dokunma).
- Teknikler (otomasyon, wordlist, keşif) sadece iki yerde serbest: ① bug bounty programının scope'u içinde, ② kendi lab'ımda / PortSwigger'da.
- "IP ban yemeyeyim" dürtüsü bir uyarı işaretidir: izinli yerde kimliğini gizlemene gerek yoktur, çünkü davet edilmişsindir.

---

## 🔜 Sıradaki Adım
- **Burp Suite kurulumu ve ilk kullanım** (Kali VM üzerinde)
- Ardından PortSwigger Web Security Academy'de **ilk IDOR lab'ını elle sömürmek**

---

## 📝 Kendi Notlarım
*(Buraya kendi eklemelerini, takıldığın yerleri, "aha!" anlarını yaz)*

