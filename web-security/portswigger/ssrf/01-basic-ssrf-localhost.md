# SSRF Lab #1: Basic SSRF against the local server — 02.11.2026

> Yeni sınıf: SSRF. Dördüncü büyük zafiyet türü. Girdi bu sefer bir sorguya/sayfaya değil, sunucunun yapacağı bir **isteğin hedefine (URL)** düşüyor — sunucuyu, senin erişemeyeceğin yerlere senin adına istek atmaya kandırıyorsun.

**Lab:** PortSwigger → SSRF → *Basic SSRF against the local server* (Apprentice)
**Görev:** Admin paneline (sadece yerelden erişilebilen) SSRF ile ulaş, carlos'u sil.

## SSRF nedir (analoji)
Binanın (iç ağ) dışındasın, kapı kilitli. İçeride her yere gidebilen güvenilir bir **posta görevlisi** (sunucu) var. Ona not uzatıyorsun: "şu adrese git, getir." Görevli içeride güvenilir olduğu için gidiyor. Sen giremedin ama **görevli aracılığıyla erişilemeze eriştin.**

## Enjeksiyon noktası
- Ürün sayfasındaki **"Check stock"** butonu arka planda bir URL'ye istek atıyor. Burp'te yakalandı; gövdede URL taşıyan parametre:
  ```
  stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
  ```
- Bu parametrenin değeri komple bir URL → sunucu bu URL'ye istek atıyor → değiştirirsek sunucuyu başka yere yönlendiririz.

## Çözüm
1. `/admin` tarayıcıdan denendi → erişilemedi (admin paneli **sadece yerelden/localhost'tan** açık, dışarıdan yetki yok).
2. Ama sunucu localhost'tur. `stockApi` değerini sunucunun kendi iç adresine çevir:
   ```
   stockApi=http://localhost/admin
   ```
   → response'ta admin panelinin HTML'i geldi (dışarıdan göremediğin sayfa artık elinde).
3. HTML'deki silme yolunu kullan:
   ```
   stockApi=http://localhost/admin/delete?username=carlos
   ```
   → carlos silindi → **Solved.** ✅

## Mantık — XSS/SQLi ile ortak kök, farklı sink
- Ortak kök (tüm injection ailesi): **uygulama doğrulanmamış girdiyi hassas bir işlemde kullanıyor.**
- Değişen = girdinin aktığı **sink**:
  - SQLi → SQL sorgusu (hedef: DB)
  - XSS → sayfa/HTML (hedef: kurbanın tarayıcısı, client-side)
  - **SSRF → sunucunun yapacağı isteğin hedefi (hedef: sunucu + iç ağ)**
- `http://localhost/admin/delete?...` **tam bir payload'dır** — fark "payload var/yok" değil, payload'ın neyi kontrol ettiği (isteğin hedefi).
- SSRF'in imzası: sunucuyu bir **proxy** gibi kullanıp iç ağa / erişilemeze ulaşmak. "localhost"/`127.0.0.1`, iç IP'ler (`192.168.x`,`10.x`), cloud metadata (`169.254.169.254`) klasik hedefler.

## 🛡️ Nasıl kapatılır (remediation)
- **Allowlist:** kullanıcıdan gelen URL'nin host/şemasını, izin verilen sabit bir listeye göre doğrula; serbest URL kabul etme.
- **İç adresleri engelle:** `localhost`/`127.0.0.1`/`::1`, özel IP aralıkları (`10/8`, `172.16/12`, `192.168/16`), `169.254.169.254` (metadata) reddedilsin. (DNS rebinding/redirect ile atlatmaya karşı: isteği atmadan önce **çözümlenen IP'yi** kontrol et, redirect'leri takip etme.)
- **Tasarım:** mümkünse kullanıcıya hiç tam URL verdirme; sadece bir ID/seçenek al, URL'yi sunucu kendi kursun.
- **Yanıtı döndürme:** fetch sonucunu kullanıcıya ham göstermemek bilgi sızıntısını azaltır (tek başına çözüm değil).

## Zincir bağı
- SQLi/XSS'te sink sorgu/sayfaydı; SSRF'te sink "isteğin hedefi". Aynı "doğrulanmamış girdi" hastalığı, yeni bir yüzey.
- Sıradaki SSRF adımları: iç ağ taraması (başka host/port'lara erişim), filtre atlatma (localhost engelliyse alternatif gösterimler).
