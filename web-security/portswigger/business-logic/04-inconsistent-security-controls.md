# Business Logic #4: Inconsistent security controls

- **Seviye:** Practitioner
- **Sınıf:** Business Logic — tutarsız güvenlik kontrolü
- **Yöntem:** Körleme (saf mantık, otomasyon yok)
- **Araç:** Tarayıcı + lab email client

---

## Zafiyet

Aynı güvenlik kuralı (şirket domaini `@dontwannacry.com` zorunluluğu) **bir giriş noktasında
uygulanıyor, başka bir noktada unutulmuş.** Kural **kayıtta** var, **e-posta değiştirmede** yok.

### Çocuğa anlatır gibi
Binaya girişte "sadece şirket çalışanı" kimlik kontrolü var. Ama içeride bir odaya geçerken kimse
bakmıyor. Sen dışarıdan girip o odaya rahatça geçiyorsun — kural **bir kapıda** var, **diğerinde** yok.

## Çözüm

1. Admin paneli sadece **`@dontwannacry.com`** e-postalı kullanıcılara açık.
2. Kendi e-postamla **normal kayıt** oldum (lab email client ile onay linkine tıklayıp aktifleştirdim).
3. Giriş yaptım → hesapta **e-postayı değiştirme** işlevi var.
4. E-postamı **`test@dontwannacry.com`** yaptım — bu işlev domaini **kontrol etmiyor.**
5. Artık "şirket çalışanı" sayıldım → **admin paneli açıldı** → carlos silindi → **çözüldü.**

## Kök neden

> Güvenlik kontrolü (domain doğrulaması) **tutarsız** uygulanmış: kayıtta zorlanıyor ama
> e-posta değiştirmede zorlanmıyor. Saldırgan, **korumasız kapıdan** (e-posta değiştirme) girer.

İkinci bir tasarım hatası: **yetkiyi e-posta domaininden çıkarmak** zaten kırılgan. Admin olmak
"e-posta @şirket mi?" ile değil, açık bir rol ile belirlenmeli.

## Business logic refleksi (bu lab'ın verdiği)

> **"Aynı kural HER yerde zorlanıyor mu?"** Bir kuralı bir akışta görünce, aynı veriyi değiştiren
> **diğer akışları** ara — biri korumasızsa oradan gir. Kayıt / profil güncelleme / API / şifre
> sıfırlama... aynı alanı değiştiren her yol ayrı kontrol edilmeli.

## Remediation (defender)

1. **Tutarlılık:** Aynı güvenlik kuralını **her giriş noktasında** uygula — domain kontrolü sadece
   kayıtta değil, e-posta değiştirmede (ve her yerde e-posta set edilebiliyorsa orada da) olmalı.
2. **Değişince yeniden doğrula:** E-posta değişince yeni adrese **onay** gönder; onaylanmadan
   geçerli sayma.
3. **Yetkiyi domainden çıkarma:** Admin yetkisi **rol tabanlı (RBAC)** verilmeli; e-posta domaini
   kimlik/yetki kanıtı olmamalı.
4. **Merkezi kontrol:** Güvenlik kararını tek bir yerde topla (her akış kendi kontrolünü
   kopyalamasın → biri unutulur).

## Büyük resim bağı

- BL #1: sunucu **price**'a güvendi.
- BL #2: **negatif quantity** (alt sınır).
- BL #3: **integer overflow** (üst sınır).
- BL #4 (bu): **tutarsız kontrol** — değere değil, **kuralın eksik uygulanmasına** dayalı.
  Aynı aile: uygulamanın bir varsayımını (kural her yerde var) kır.

## Köşe taşı

> Bir güvenlik kuralı bir kapıda varsa, **aynı veriyi değiştiren her kapıyı** kontrol et — biri
> korumasızsa tüm kural anlamsızdır. Ve e-posta domaini gibi dolaylı sinyaller yetki kanıtı olamaz.
