# İlgi Pusulası — hangi alana yatkınım?

> Amaç: "her şeye ilgim var" belirsizliğini, zamanla **keyif verisiyle** çözmek. Zorla alan seçmek yok; neyi yaparken "aktığım" biriktikçe uzmanlaşma kendini gösterir. Her lab sonrası kısa bir keyif notu; her milestone'da deseni okuruz.

## Siber güvenlik alanları (hepsi aynı temeli paylaşır)
- Offensive web / bug bounty · Network/infra pentest · Red team · Exploit dev / vuln research · Security tooling & automation · AppSec / secure code review · Blue team / DFIR

## İlk sinyal — Gün 27 (01.11.2026)
- "Erişilemez sanılan şeylere **erişmek**, sistemlere **girmenin yolunu bulmak** çok havalı geliyor." → *içeri girme / erişim* hazzı.
- Blind SQLi'de en sevdiği: **şifreyi ele geçirmek** (avın/trophy'nin kendisi).
- Angarya hissettiren kısım: yok.
- **Yorum:** sinyal **offensive/exploitation** tarafını işaret ediyor (bug bounty / pentest / red team'in "kırıp girme" hazzı) — CV'den beklenen tooling/AppSec tarafından farklı. Tek veri noktası, erken; biriktirince netleşecek.

## Keyif günlüğü (her lab: 1–5 + bir cümle)
| Lab | Keyif | En sevdiğim / en sıkıcı an |
|-----|:-----:|-----------------------------|
| Blind SQLi (genel: responses/errors/time) | 5 | Sevdi: şifreyi ele geçirme avı + script yazma. Angarya: yok. |
| SSRF #1 (basic, localhost) | 4 | Sevdi: sunucuyu kandırıp erişilemez admin paneline ulaşmak ("erişilemeze erişmek"). |
| SSRF #2 (iç ağ tarama) | 4 | Uğraştırdı (Intruder engeli→Python, anomali avı) ama keyifliydi; iç ağda gizli host bulmak. |
| SSRF #3 (blacklist atlatma) | 4 | "Yasak olanın cazibesi" — filtreyi kılık değiştirerek aşmak; zor sanıp hızlı çözmek. |
| SSRF #4 (whitelist, Expert) | 4 | İlk Expert; parser disagreement. Çok zordu, tek başına bulamadı ama mantığı sevdi. |
| Command Injection #1 (simple) | 5 | İLK 5. Sunucuda komut çalıştırıp dosya sistemine daldı (cd/ls). "Bayıldım." Zirve offensive keyfi. |
| Access Control (IDOR+pw disclosure) | 4 | IDOR zincirini kendi başına kurdu (inspect→şifre→admin). Gerçek-hedef giriş sınıfı. |
| Authentication #1 (username enum + brute) | 4 | Python brute-force "havalı ve zevkli". İki aşamalı kendi scripti. Takıntı: file-I/O + param swap. |
