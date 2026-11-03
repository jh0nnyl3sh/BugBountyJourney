# SSRF Lab #2: Basic SSRF against another back-end system — 03.11.2026

> SSRF ile iç ağ keşfi. Bu sefer hedef localhost değil, iç ağdaki BAŞKA bir makine — ve adresini bilmiyorsun. Çözüm: aralığı tara, gürültü içinden anomaliyi (admin host) bul.

**Lab:** PortSwigger → SSRF → *Basic SSRF against another back-end system* (Apprentice)
**Görev:** İç ağdaki admin arayüzünü (192.168.0.0/24, port 8080) bul, carlos'u sil.

## Önceki lab'dan farkı
- Lab #1: hedef **localhost** (sunucunun kendisi), adresi biliyordun.
- Lab #2: hedef iç ağdaki **başka bir host**, hangi IP olduğu **bilinmiyor** → bulman gerek.

## Enjeksiyon noktası
`stockApi` değeri iç ağı gösteriyordu: `http://192.168.0.1:8080/product/stock/check?...`. Demek iç sistemler `192.168.0.0/24`, port `8080`. Admin arayüzü bu aralıkta bir X'te, `/admin` yolunda.

## Tarama — tool sınırlıysa script yaz
- **Burp Intruder** denendi → Community (demo) sürümü Intruder'ı kasıtlı yavaşlatıyor, 255'lik tarama pratik değil. **Ders: aracın sınırlandığında kendi aracını yaz.**
- Python ile Check stock isteği taklit edildi; `stockApi` 1–255 arası `http://192.168.0.X:8080/admin`'e ayarlandı. requests `data={}` ile encode'u hallediyor; dead host'larda takılmamak için `timeout`.

## Anomali analizi (asıl beceri)
Yanıtlar `(status, uzunluk)` olarak gruplandı:
| Grup | Adet | Anlamı |
|------|:----:|--------|
| (500, 2454) | 253 | Ölü IP'ler — bağlanılamadı, hep aynı hata. **Gürültü/baseline.** |
| (400, 19) | 1 (.1) | Stok servisi var ama `/admin` geçersiz → işe yaramaz. |
| **(200, 3245)** | **1 (.132)** | **200 + tam HTML = admin arayüzü.** Sinyal bu. |

255 yanıtı tek tek okumadık; **gürültü hep aynıdır, anomaliyi (farklı status/uzunluk) yakalarsın.** (ffuf/gobuster'da da aynı mantık: yanıtı boyut/koda göre filtreleyip farklı olanı bulmak.)

## Son darbe
```
stockApi=http://192.168.0.132:8080/admin/delete?username=carlos
```
→ carlos silindi → **Solved.** ✅

## 🛡️ Nasıl kapatılır (remediation)
- **Allowlist:** `stockApi` serbest URL almamalı; yalnızca bilinen stok servis host/portuna izin ver, iç IP aralıklarını reddet.
- **Çözümlenen IP'yi kontrol et** (DNS rebinding/redirect atlatmasına karşı); redirect takip etme.
- **Ağ segmentasyonu:** admin arayüzü, uygulama sunucusunun erişebildiği bir iç ağda açıkta durmamalı; hassas iç servisler erişilebilir olsa bile **kimlik doğrulaması** istemeli ("iç ağdayım = güvenilirim" varsayımı yanlış).
- Hata yanıtlarını generic tut (tarama anomalilerini zayıflatır; tek başına çözüm değil).

## Zincir bağı
- Lab #1 localhost'a erişti; #2 iç ağ keşfi ekledi (host bulma + anomali tarama).
- "Aralık tara + anomaliyi bul" refleksi, recon/fuzzing'in temeli — ileride content discovery, port/host keşfi hep bu mantık.
