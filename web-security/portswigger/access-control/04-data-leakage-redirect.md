# 9. IDOR Lab #4: Data leakage in redirect — 11.10.2026

> Dördüncü çözülen lab. "Neden hep Burp?" sorusunun iliklerde hissedildiği lab.

**Lab:** PortSwigger → Access control → *User ID controlled by request parameter with data leakage in redirect* (Apprentice)
**Görev:** `carlos`'un API key'ini bul ve gönder.
**Verilen hesap:** `wiener:peter`

## Tuzak
- `id=carlos` denince tarayıcı seni **login sayfasına** atıyor → tarayıcıda hiçbir şey görünmüyor, "olmadı" sanıyorsun.
- Gerçekte: sunucu carlos'un sayfasını **tam hazırlıyor** (API key yanıtın gövdesinde), *sonra* yanıta bir **302 redirect** (login'e) ekliyor.
- Tarayıcı redirect'e itaat edip gövdeyi atlıyor → veri sızmış ama göremiyorsun.

## Çözüm
1. `wiener:peter` ile giriş, URL'de `id=wiener`.
2. `id=carlos` denendi → tarayıcı login'e attı (iki kez). "Boş" göründü.
3. Burp HTTP history / Intercept ile istek yakalandı, **ham yanıta** bakıldı.
4. 302 redirect'in altındaki **gövdede carlos'un API key'i** duruyordu.
5. Key alındı, gönderildi → **Solved.** ✅
6. (Doğrulama merakı) Forward'lanınca yine login'e düştü → yani veri önce üretiliyor, redirect sonra ekleniyor.

## Kök sebep
- Yine **authorization eksikliği**, ama özel biçimiyle: kontrol **çok geç** yapılıyor.
- Sunucu önce veriyi üretip *sonra* "yetkin yok" diyor. Veri bir kez yanıt gövdesine girdi mi, redirect onu kurtarmaz.
- Doğrusu: **veriyi üretmeden önce** yetki kontrolü yapılmalı.

## Altın ders (neden Burp?)
- **Tarayıcı yönlendirmeleri (redirect) senin için takip eder ve yanıt gövdelerini gizleyebilir.**
- Burp ham yanıtı olduğu gibi tutar. Avcı, tarayıcının "boş" demesine asla güvenmez; ham yanıta bakar.
- Pratik: istek → Burp HTTP history / Repeater → redirect'i takip etmeden ham gövdeyi incele.

## Süreç notu
İpucu bilerek okunmadan "kör" denendi; takılınca Repeater sağ-tık menüsünden kendi kendine keşfedildi. Takılıp debelenmek, öğrenmenin ta kendisi.
