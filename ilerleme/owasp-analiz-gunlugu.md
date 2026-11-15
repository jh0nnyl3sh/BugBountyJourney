# OWASP Top 10 — Analiz Günlüğü

> Haftada bir (Pazar) bir OWASP kategorisini masaya yatırıyoruz: ne olduğu, alt türleri,
> PortSwigger konularıyla haritası, remediation çerçevesi — sonra o aileye uygun lab(lar).
> Amaç: 200 tekniği ezberlemek yerine 10 **aile** öğrenmek; her yeni zafiyet bir aileye otursun.

---

## #1 — A01: Broken Access Control — 15.11.2026

**Çocuğa anlatır gibi:** Otel oda anahtarları. Access control = "bu anahtar sadece bu kapıyı açsın".
Broken = anahtarların karışması (seninki başkasının/müdürün odasını açıyor), ya da kapıda kilit
yerine sadece "girmeyin" tabelası (gizlilik ≠ güvenlik).

**Üç ayak:** Authentication (kimsin? → A07) · **Authorization (ne yapabilirsin? → A01 bu)** ·
Enforcement (kuralların zorlanması).

**Alt türler:**
- **IDOR:** `?id=123` → `?id=124`, başkasının verisi.
- **Vertical priv esc:** normal kullanıcı → admin işlevi.
- **Horizontal priv esc:** kullanıcı A → kullanıcı B'nin hesabı.
- **Missing function-level access control / forced browsing:** `/admin`'i elle yaz, korunmuyorsa girer.
- **Parameter-based:** `role=user`→`admin`, `isAdmin=false`→`true`.

**Neden #1:** Her uygulamada "kim neye erişir" mantığı var ve kolay kırılır; impact yüksek
(veri, admin, hesap ele geçirme); bulması **düşünmekle** olur (araçla değil) → Uğur'un keyif aldığı iş.

**PortSwigger haritası:** Access Control / IDOR sınıfı = A01. (8 lab + bugün re-solve.)

**Remediation çerçevesi:**
1. Deny by default.
2. Server-side enforce (butonu gizlemek yetmez; endpoint yetkiyi kontrol etmeli).
3. Object ownership kontrolü (bu kayıt gerçekten bu kullanıcının mı?).
4. Role/yetkiyi istekten alma; oturumdan türet.
5. Güvenlik ≠ gizlilik (`/admin`'i saklamak koruma değil).

**Pekiştirme lab'ı:** AC #5 *Unprotected admin functionality* re-solve — `robots.txt` (Info
Disclosure) → `/administrator-panel` → access control yok → carlos silindi. **İki OWASP ailesi
zincirlendi:** Info Disclosure kapıyı buldurdu, Broken Access Control içeri soktu.

**Günün dersi:** Gerçek saldırı tek zafiyet değil, **zincirdir.** Recon (sızıntı) → erişim (yetki açığı) → impact.
