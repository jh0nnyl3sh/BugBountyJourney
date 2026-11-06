# Command Injection Lab #1: simple case — 06.11.2026

> Yeni sınıf: OS Command Injection — en sert sink. Girdi, sunucuda çalışan bir işletim sistemi (shell) komutuna düşüyor; araya kendi komutunu sıkıştırınca sunucu onu da çalıştırıyor (RCE).

**Lab:** PortSwigger → OS command injection → *simple case* (Apprentice)
**Görev:** Sunucuda `whoami` çalıştırıp çıktısını görmek.

## Analoji
Naif mutfak robotu: siparişi fişe yazarsın, robot fişteki *her şeyi* yapar. "1 pizza **; arka kapıyı aç**" yazarsan ikisini de yapar. Araya ekstra emir sızdırdın.

## Enjeksiyon noktası + çözüm
"Check stock" özelliği arka planda bir shell komutu kuruyor ve parametreleri girdiden alıyor:
```
stockreport.sh <productId> <storeId>
```
Payload (`storeId`):
```
productId=13&storeId=1;whoami
```
Çalışan komut: `stockreport.sh 13 1;whoami` → `;` komutu ayırır → `whoami` da çalışır → çıktı: **`peter-GLVQ2U`** → **Solved.** ✅

## Komut ayırıcılar (zincirleyiciler)
`;` · `&` · `|` · `&&` · `||` · `` `komut` `` · `$(komut)` · yeni satır. Komutu kesip araya yeni komut sokar.

## Keşif (hands-on)
Çözdükten sonra içeri dalındı: `storeId=1;cd ..; ls` → `/home` altında başka kullanıcılar (`carlos`, `elmer`, `user`, `peter-GLVQ2U`, `install`). Yani komut çalıştırma = sunucu dosya sistemine erişim. **Post-exploitation = Linux/bash akıcılığı.**

## Neden en yüksek etki
Girdi doğrudan **sunucunun shell'ine** akıyor → **RCE (remote code execution).** Sunucu olarak dosya okursun, veri sızdırırsın, değişiklik yaparsın. SQLi DB'yi, XSS tarayıcıyı hedeflerken, command injection **sunucunun kendisini** verir.

## ⚖️ Etik çizgi (RCE'de kalınlaşır)
- **Lab / kendi sistem:** sonuna kadar dal.
- **Gerçek hedef:** etkiyi **en zararsız** komutla kanıtla (`whoami`, `id`, `hostname`). ASLA `rm -rf`, veri silme/değiştirme, servis bozma; scope dışına çıkma.
- "Silebilirdim" demek yeterli kanıt; silmek suç. Beyaz şapkayı suçludan ayıran = "yapabilmek" ≠ "yapmak".

## 🛡️ Nasıl kapatılır (remediation)
- **En iyisi:** kullanıcı girdisiyle hiç OS komutu çağırma; iş için **güvenli dil API'leri** kullan (ör. dosya/işlem için kütüphane fonksiyonları, shell değil).
- Mecbursan: **shell'i baypas et** (komutu argüman dizisiyle çağır, `shell=True`/string birleştirme YOK), ve girdiyi **allowlist** ile sıkı doğrula (ör. sadece rakam).
- Girdiyi "temizlemeye" (metakarakter blacklist'i) güvenme — kırılgan (bkz. SSRF blacklist dersi).

## Zincir bağı
- Ortak kök yine: doğrulanmamış girdi bir sink'e. Sink'ler: SQLi→sorgu, XSS→HTML, SSRF→istek URL'si, **Command Injection→OS shell** (en sert).
- Sıradaki: kör (blind) command injection — çıktı ekranda görünmediğinde (SQLi blind'ının kardeşi: output yönlendirme, time delay, OAST).
