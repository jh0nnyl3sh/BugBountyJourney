# 2. HTTP İsteğinin Anatomisi

> Bütün avcılık hayatın bu küçük metin parçasının üstünde dönecek.

Bir HTTP isteği aslında düz metindir:

```
GET /profil HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
```

Satır satır, her biri bir avcı için ipucu:

- **İlk satır** üç şey söyler: **metot** (`GET`) + **yol/path** (`/profil`) + **versiyon** (`HTTP/1.1`). İsteğin kalbidir.
- **Host** — hangi siteyle konuşuyorum (tek sunucuda yüzlerce site olabilir).
- **User-Agent** — kendimi nasıl tanıtıyorum ("ben Chrome'um"). *Dikkat: bu satır yalan söyleyebilir, avcılıkta işe yarar.*
- **Cookie** — kimliğim. Session ID burada taşınır; sunucu beni her istekte bundan tanır.

## HTTP Metotları

- `GET` → veri getir
- `POST` → veri gönder (örn. form)
- `PUT` → güncelle
- `DELETE` → sil

**Avcı sorusu:** "Bu sayfa `GET` bekliyor ama ben `DELETE` gönderirsem ne olur? Sunucu beni durdurur mu, yoksa gerçekten siler mi?"

## Kilit Kavram: HTTP durumsuzdur (stateless)

Sunucu seni hatırlamaz; her istek sıfırdan gelir, sanki seni ilk kez görüyormuş gibi. "Giriş yaptım" durumunun nasıl korunduğu bir sonraki konunun (Oturum Mantığı) meselesidir — ve en büyük zafiyet sınıflarından biri tam oradaki çatlaklarda yaşar.
