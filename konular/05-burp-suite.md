# 5. Burp Suite — İsteği Havada Yakalamak

> Bütün web "istek git, yanıt gel" döngüsünde döner; avcılığın kalbi o isteği ele geçirip onunla oynamaktır. Burp Suite tam olarak bunu yapan alettir.

## Burp nedir?

Tarayıcı ile sunucu arasına oturan bir **proxy** (ara durak). Normalde istek göz açıp kapayana kadar sunucuya gider; Burp araya girip "sunucuya gitmeden önce bana göster" der. İsteği görür, durdurur, değiştirir, yeniden gönderirsin.

## Başta tanıyacağımız iki araç

- **Proxy** → İstekleri havada yakalayıp durduran kısım (kalp burası).
- **Repeater** → Yakalanan bir isteği alıp defalarca değiştirip tekrar gönderdiğin tezgah (IDOR denemeleri burada: 1004 → 1005 → 1006...).

> Intruder, Scanner, Decoder vb. ileride, ihtiyaç oldukça açılacak. Hepsini birden öğrenmeye çalışmak sindirerek ilerleme ilkesine aykırı.

## Kurulum / çalıştırma notları (Kali + UTM)

- Burp çoğu Kali kurulumunda hazır gelir (Community Edition). Açılışta: **Temporary project** + **Use Burp defaults** → Start.
- Gömülü tarayıcı (Open Browser) Kali'de **root yüzünden** açılmayabilir. Çözüm: Settings → `browser` ara → "Allow Burp's browser to run without a sandbox". Ya da Firefox + FoxyProxy kullan (tercih edilen, çünkü "gerçek" yöntem).

## Firefox + FoxyProxy ile bağlama

1. FoxyProxy'de profil: host `127.0.0.1`, port `8080` (Burp'ün varsayılan listener'ı).
2. FoxyProxy = "av moduna geç / normale dön" anahtarı. İş bitince kapat, yoksa normal gezinti takılır.
3. Burp tarafında **Proxy → Proxy settings** altında listener'ın `127.0.0.1:8080 running` olduğunu doğrula.

## İlk yakalama (kanıtlanmış akış)

- FoxyProxy açık + Burp'te `Intercept is on`.
- Firefox'ta `http://example.com` → sayfa **takılır** (iyiye işaret: istek Burp'te durduruldu).
- **Proxy → Intercept**'te ham istek görünür:
  ```
  GET / HTTP/1.1
  Host: example.com
  User-Agent: Mozilla/5.0 ... Firefox/140.0
  ...
  ```
  - Yol `/` = sitenin **kök dizini** (ana sayfa). `/profil` isteseydin `/profil` yazardı.
  - Sağdaki **Inspector** paneli isteği parçalara ayırır (headers, cookies, query params). example.com'da cookie=0, çünkü giriş yapmadık → kimliğimiz yok.
- **Forward** → istek sunucuya gider, sayfa yüklenir. (**Drop** → isteği çöpe atar, hiç gönderilmez.)

## HTTPS için sertifika

Burp sadece http'yi düz yakalar; https şifreli olduğu için Firefox'un Burp'e güvenmesi gerekir.

1. FoxyProxy açık, Intercept **off**.
2. Firefox'ta `http://burp` → **CA Certificate** indir (`cacert.der`).
3. Firefox → Settings → Privacy & Security → Certificates → View Certificates → **Authorities** → **Import** → `cacert.der`.
4. "Trust this CA to identify websites" işaretle → OK.
5. Test: https bir siteye gir, uyarı yoksa tamam.

## Sözlük

- **Intercept on/off** → yakalamayı aç/kapat.
- **Forward** → isteği olduğu gibi (ya da düzenleyip) sunucuya yolla.
- **Drop** → isteği hiç gönderme.
- **HTTP history** → yakalanıp geçen tüm isteklerin kaydı (Intercept kapalıyken bile dolar).
