# Information Disclosure #1 — Information disclosure in error messages

- **Seviye:** Apprentice
- **Sınıf:** Information Disclosure (Bilgi Sızıntısı)
- **Yöntem:** Körleme (link verildi, tek başıma çözüldü)
- **Araç:** Tarayıcı (Burp hazırdı ama gerek kalmadı)

---

## Zafiyet nedir?

Uygulama, kullanıcıya **gösterilmemesi gereken hassas bilgiyi** sızdırıyor. Burada sızan şey
bir **hata mesajı** — ama sıradan değil, tam bir **stack trace**: kullanılan teknoloji, framework
ve **tam sürüm numarası** açığa çıkıyor.

Bu bir **injection değil, keşif (recon)**. Bir şey enjekte etmiyoruz; uygulamayı hata vermeye
zorlayıp **sızan bilgiyi topluyoruz**. Bu bilgi çoğu zaman bir sonraki saldırının ilk basamağı:
sürümü öğren → o sürümün bilinen CVE'sini bul → exploit et.

## Analoji

Bir dükkânın tezgâhında, müşterinin görmemesi gereken **arka ofis evrakları** açıkta duruyor.
Tek bir yanlış soru soruyorsun ve kasiyer panikle tüm iç prosedürü (hangi sistem, hangi sürüm)
masaya döküyor. Information disclosure tam bu.

## Çözüm adımları (kendi anlatımımdan)

1. Lab'ı açtım, sade bir sayfa: sadece **Home** butonu ve ürün bağlantıları.
2. Kaynağı kurcaladım — HTML **comment** (yorum satırı) arıyordum, bu lab'da yoktu.
3. URL'de ürün bağlantılarının `productId=1` gibi **sayısal** bir parametre taşıdığını gördüm.
4. Bu parametreye **sayı yerine string** (`a`) girdim: `.../product?productId=a`
5. Backend `Integer.parseInt("a")` yapmaya çalıştı → **patladı** → tüm stack trace sayfaya düştü.
6. Trace'in **en dibindeki** satırda sürüm vardı → lab'a gönderdim → **çözüldü**.

## Sızan kritik bilgi

```
Internal Server Error: java.lang.NumberFormatException: For input string: "a"
	at java.base/java.lang.Integer.parseInt(...)
	...
Apache Struts 2 2.3.31
```

- **Teknoloji:** Java (`java.lang`, `ThreadPoolExecutor` → stack trace'ten okunur)
- **Framework + sürüm:** **Apache Struts 2 — 2.3.31**

> Not: Struts 2'nin bu sürüm aralığı, gerçek dünyada kritik RCE CVE'leriyle (ör. S2 serisi)
> anılır. Yani "sadece bir sürüm numarası" değil — saldırgan için **doğrudan bir exploit kapısı**.

## Neden çalıştı? (kök neden)

`productId` sayı bekliyor ama uygulama **girişi doğrulamadan** doğrudan `parseInt`'e verdi.
Fırlayan exception **yakalanmadı** ve **production'da verbose hata** açık olduğu için ham trace
kullanıcıya gösterildi. İki hata üst üste: **input validation yok** + **error handling yok**.

## Remediation (defender tarafı)

1. **Generic error messages:** Kullanıcıya asla stack trace gösterme. "Bir hata oluştu" yeterli;
   detay yalnızca sunucu **log**'una gider.
2. **Production'da debug/verbose kapalı:** `devMode=false` (Struts), `customErrors=On` (ASP.NET),
   `debug=False` (Flask/Django). Ortamlar arası (dev/prod) ayrımı net olmalı.
3. **Input validation:** `parseInt`'e gitmeden tipi doğrula; hatalı girdiyi exception fırlatmadan
   **temiz reddet** (400 + generic mesaj).
4. **Sürümü güncel tut:** Sızıntıyı tamamen kapatamasan da, bilinen açığı olan eski sürümleri
   (2.3.31) güncelleyerek CVE kapısını kapat.

## Zincir bağı (önceki konularla)

- **Injection** (SQLi/XSS/Command Injection): biz bir şey **enjekte** ediyorduk.
- **Information Disclosure:** burada bir şey enjekte etmiyoruz, **sızanı buluyoruz** → saf recon.
- **SSRF → iç ağ tarama** ile aynı ruh: önce *haritayı* çıkar, sonra *vur*.
- Bu bulgu, **recon & metodoloji** modülünün kalbi: sürüm/teknoloji tespiti her gerçek testin
  ilk adımı. Lab'daki `productId=a` hamlesi, sahada "error'a zorla, sızıntıyı oku" refleksinin ta kendisi.

## Köşe taşı

> Sayı beklenen her parametre, bir **hata tetikleyicisidir**. String/özel karakter sok, dönen
> hatayı **birebir oku** — framework, sürüm, dosya yolu, DB adı hepsi orada olabilir.
