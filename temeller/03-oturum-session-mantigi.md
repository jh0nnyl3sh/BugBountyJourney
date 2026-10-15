# 3. Oturum (Session) Mantığı — "Sunucu beni nasıl tanıyor?"

> HTTP durumsuzdur; sunucunun hafızası yok. Peki "giriş yaptım" durumu nasıl korunuyor?

## Bileklik benzetmesi

Bir kafeye girip giriş yapıyorsun. Ama garsonun hafızası yok, her masaya gelişinde seni ilk kez görüyor. Çözüm: giriş yapınca eline bir **bileklik** takıyor, üstünde bir numara var (`abc123`). Her siparişte bileği gösteriyorsun, garson numaraya bakıp seni tanıyor.

O bileklik = çerezdeki **session ID**.

## Teknik karşılığı

1. Kullanıcı adı + şifreyle giriş yaparsın (bir `POST` isteği).
2. Sunucu doğrularsa sana **rastgele ve tahmin edilmesi zor** bir session ID üretir (`session=a9f3k2l8x7...`).
3. Bunu sana **`Set-Cookie`** yanıtıyla gönderir. *(Set-Cookie = sunucudan sana geliş yönü)*
4. Tarayıcın bu çerezi saklar ve bundan sonra **her istekte** otomatik geri yollar. *(`Cookie` = senden sunucuya gidiş yönü)*
5. Sunucu kendi tarafında "`a9f3k2l8x7` = bu kullanıcı" kaydını tutar.

Böylece durumsuz protokolün üstüne çerezle sahte bir "hafıza" kurulur.

> **Yön farkı önemli:** `Set-Cookie` bileklik takılırken (geliş), `Cookie` her gösterdiğinde (gidiş).

## Oturum zafiyetlerinin doğduğu 4 soru

Cevap "hayır" ise orada zafiyet var:

1. Session ID gerçekten **tahmin edilemez** mi? (yoksa *session prediction*: `1, 2, 3...` sırayla veriliyorsa başkasının oturumu çalınabilir)
2. Çerez **çalınabilir** mi? (*session hijacking*: başkasının ID'sini ele geçirip onun gibi davranmak)
3. Çıkışta (logout) **gerçekten iptal** oluyor mu? (zayıf oturum sonlandırma)
4. **Koruma etiketleri** var mı? (`HttpOnly`, `Secure`, `SameSite` — ileride tek tek işlenecek)

> **Not:** Tehlike sadece "ben kaptırırsam" değil; bazen garson bilekliği en baştan kötü tasarlamıştır (tahmin edilebilir numara, iptal etmeyen çıkış...).
