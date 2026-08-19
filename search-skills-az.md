# TryHackMe - Search Skills | Azerbaycan dilinde Izahli Yazisi

> **Mənbə:** https://tryhackme.com/room/searchskills
> **Cetinlik:** Easy | **Muddet:** ~30 deq
> **Məqsəd:** İnternetde effektiv axtarış etmeyi, xususi axtarış motorlarini ve texniki senedleri istifadə etmeyi oyrənmək.

---

## Mundəricat

1. [Task 1 - Giris](#task-1)
2. [Task 2 - Axtariş Nəticələrini Qiymətləndirmək](#task-2)
3. [Task 3 - Axtariş Motorlari ve Operatorlar](#task-3)
4. [Task 4 - Xususi Axtariş Motorlari](#task-4)
5. [Task 5 - Zəifliklər ve Exploitlər](#task-5)
6. [Task 6 - Texniki Senedler](#task-6)
7. [Task 7 - Sosial Media ve OSINT](#task-7)
8. [Task 8 - Nəticə](#task-8)

---

## Task 1 - Giris

### Niyə Axtariş Bacariği Vacibdir?

"learn hacking" ifadesini Google-da axtarsan milyardlarla nəticə gelinr. Bu nəticələrin hamisi doğru ve etibarli deyil — coxu yanlis, kohnəlmiş ve ya sadece yanlış yonlendirici ola bilər.

**Kibertəhlukəsizlikde bu o deməkdir:**

```
Bir zaiflik tapdın  ->  CVE kodu nədir?
Exploit var mi?     ->  Harada baximaq lazimdir?
Bu alət nə edir?    ->  Rəsmi sənədi haradadir?
Şirkətin işçisi kim? ->  OSINT ilə necə tapmali?
```

Sürətli ve düzgün məlumat tapmaq bacarığı kibertəhlükəsizlikde **texniki biliklər qədər vacibdir.**

Bu labda öyrənəcəklər:

| Bacariq | Nə deməkdir? |
|---------|--------------|
| **Etibarliligi qiymətləndirmək** | Mənbənin doğru olub-olmadığını müəyyən etmək |
| **Axtariş operatorlari** | Google-dan maksimum fayda almaq ucun xüsusi komandalar |
| **Xususi alətlər** | Shodan, VirusTotal, Censys kimi platformalar |
| **Texniki sənədlər** | Man pages, Microsoft Docs, rəsmi bələdcilər |
| **OSINT** | Sosial media ve aciq mənbələrdən kəşfiyyat |

---

## Task 2 - Axtariş Nəticələrini Qiymətləndirmək

### Problem: İnternetde Hər Şey Doğru Deyil

İnternetdə bir məlumat tapdıqda ona körüklü güvənmək olmaz. Xüsusilə kibertəhlükəsizlikdə yanlış məlumat **ciddi zərər** verə bilər.

### 4 Sual — Mənbəni Yoxla

Hər oxuduğun məlumat ucun bu 4 suali ver:

```
1. MƏNBƏ:      Bunu kim yazdı? Mötəbər qurum/şəxsdirmi?
2. DƏLIL:      İddiaları faktlar ve məntiqlə dəstəklənir?
3. OBYEKTİVLİK: Məlumat tərəfsizdir, yoxsa bir tərəfə xidmət edir?
4. TƏSDİQ:     Başqa etibarli mənbə eyni şeyi deyirmi?
```

### Nümunə: Yaxşı vs Pis Mənbə

```
Kibertəhlukəsizlik ucun YAXŞI mənbələr:
  - NIST (nist.gov)
  - SANS Institute (sans.org)
  - CVE Database (cve.org)
  - Vendor rəsmi sənədlər (Microsoft Docs, Cisco Docs)
  - Red Hat Blog, Cloudflare Blog

Kibertəhlukəsizlik ucun PİS mənbələr:
  - Adsiz forumlar
  - Tarixi bəlli olmayan bloglar
  - Sosial mediada paylasilmiş "hacker tricks"
  - "Trust me bro" tipli saytlar
```

### Snake Oil nədir?

**Snake Oil** — saxta ve ya dəyərsiz kriptografik metod ve ya məhsuldur.

Tarixən Amerika-da satıcılar hər şeyi sağaldan "ilan yağı" (snake oil) satırdılar. Bu, işə yaramayan bir şeyi möcüzəvi kimi satmaq mənasına gəldi. Kriptografiyada isə **Snake Oil** — güvənli olmayan, zəif, saxta şifrələmə metodlarını bildirir.

```
Real nümunə:
"Kəsilməz şifrələmə sistemi!" deyə satılan bir proqram,
əslində çox sadə XOR şifrələməsini işlədib.
Bu "Snake Oil" kriptografiyadır.
```

---

## Task 3 - Axtariş Motorlari ve Operatorlar

### Google Sadə Axtarişdan Daha Güclüdür

Google-da xüsusi operatorlar işlədərək **çox daha dəqiq nəticə** ala bilərsən. Bunlara **Google Dork** operatorlari da deyilir.

---

### Operator 1: Dırnaq İşarəsi `"..."`

**Nə edir:** Tam ifadenin aynen tapilmasini tələb edir.

```
Adi axtariş: passive reconnaissance
  -> "passive" ve "reconnaissance" ayrı-ayrı yerlerde ola bilər

Dırnaqla: "passive reconnaissance"
  -> Məhz bu iki söz yan-yana olan nəticələr gelir
```

**Ne vaxt istifade et:**
- Dəqiq termin tapmaq istəyəndə
- Xüsusi xəta mesajını axtaranda: `"access denied to /etc/passwd"`
- CVE açıqlamasının dəqiq mətni

---

### Operator 2: `site:`

**Nə edir:** Axtarişı yalnız bir sayta məhdudlaşdırır.

```
site:tryhackme.com nmap          -> Yalnız TryHackMe-dəki nmap məqalələri
site:gov.uk cybersecurity        -> Yalnız .gov.uk domenlərindəki nəticələr
site:reddit.com kali linux tips  -> Yalnız Reddit-dəki nəticələr
```

**Pentest-də istifadəsi:**
```
site:linkedin.com "TargetCompany" engineer
-> Hədəf şirkətin LinkedIn-dəki mühəndisləri

site:github.com "TargetApp" config
-> GitHub-da hədəf tətbiqin konfiq faylları
```

---

### Operator 3: `-` (Çıxar)

**Nə edir:** Müəyyən sözü nəticələrdən çıxarır.

```
pyramids -tourism
-> Misir ehramlari haqqinda, amma turizm mövzusunda olmayan məqalələr

python -snake
-> Programlama dili Python haqqinda, ilan haqqında deyil
```

---

### Operator 4: `filetype:`

**Nə edir:** Yalnız müəyyən fayl növünü axtarır.

```
filetype:pdf cyber warfare report
-> PDF formatında kibermüharibə hesabatları

filetype:xlsx employee list 2024
-> Excel faylında işçi siyahıları (OSINT ucun vacib!)

filetype:txt password
-> .txt faylinda "password" kecən sənədlər
```

**Kibertəhlukəsizlikde niyə vacibdir:**

```
Pentest zamanı:
filetype:pdf "TargetCompany" internal
-> Hədəf şirkətin ictimai PDF-ləri (gizli olmayan, amma faydalı)

filetype:env DB_PASSWORD site:github.com
-> GitHub-da acıqda qalmış .env faylları!
```

---

### Operatorlari Birleşdirmək

Operatorlar birlikde işlədilə bilər:

```
site:tryhackme.com filetype:pdf "penetration testing"
-> TryHackMe-dəki penetrasyon testi haqqında PDF-lər

"SQL injection" site:owasp.org -forum
-> OWASP-da SQL injection, amma forum yazilari olmadan
```

---

### `ss` komutu — `netstat`-ın Varisi

Köhnə Linux sistemlərindəki `netstat` əmri artıq köhnəlmiş sayılır. Onun yerini **`ss`** əmri almışdır.

**`ss`** = **s**ocket **s**tatistics = Soket Statistikaları

```bash
netstat -tuln    # Köhnə üsul
ss -tuln         # Yeni üsul — eyni nəticə, daha sürətli

# ss nümunələri:
ss -t     # TCP soketlərini gostər
ss -u     # UDP soketlərini gostər
ss -l     # Dinləyən (listening) soketlər
ss -p     # Prosesleri gostər
ss -n     # Ünvanları ədəd kimi gostər (adla deyil)
```

---

## Task 4 - Xususi Axtariş Motorlari

Adi Google şəbəkə cihazlarını, zəiflikləri, virus heshlerini axtara bilməz. Bunun ucun **xüsusi alətlər** lazımdır.

---

### Shodan — İnternetə Bağlı Cihazların Axtarış Motoru

**Shodan** (shodan.io) — İnternetə bağlı cihazları indeksləyən axtarış motorudur. Veb saytları deyil, **cihazları** axtarır.

```
Google: "HTTP serverlər" deyə axtarirsan -> Saytlar haqqında məqalələr tapir
Shodan: "HTTP serverlər" deyə axtarirsan -> Açiq HTTP serverləri tapır!
```

**Nə tapa bilər?**

```
- Açiq portlar olan serverlər
- Xüsusi proqram versiyaları işlədən cihazlar
- Sənaye avadanlığı (SCADA sistemləri)
- Kameralar, routerlər, printerler
- Tibbi cihazlar
- Ağıllı ev cihazları
```

**Nümunə axtarışlar:**

```
lighttpd               -> lighttpd veb server işlədən bütün cihazlar
apache 2.4.1           -> Konkret Apache versiyası işlədən serverlər
country:AZ             -> Azərbaycandakı açıq cihazlar
port:22 country:DE     -> Almaniyadakı açıq SSH portları
city:Baku              -> Bakıdakı açıq cihazlar
```

**Kibertəhlukəsizlikde niyə vacibdir:**

```
Pentest zamanı (icazəli):
  - Hədəf şirkətin açıq portlarını gör
  - Köhnə, yamaqlanmamış software tap
  - Şəbəkə xəritəsi çək

Müdafiə zamanı:
  - Öz şirkətinin açıq cihazlarını yoxla
  - Gözlənilməz açıq portlari tap
```

> **QEYD:** Shodan-da nə tapılırsa, onu açıqdan istifadə etmək cinayətdir. Yalnız icazəli pentestdə istifadə et.

---

### Censys — Daha Texniki Alternativ

**Censys** (censys.io) — Shodana oxşar, amma daha çox:
- Domenler
- SSL/TLS sertifikatlar
- Açıq portlar
- Şəbəkə infrastrukturu

üzərindəki məlumatlara fokuslanır. Tədqiqatçılar ucun daha uyğundur.

---

### VirusTotal — Fayl ve URL Analizi

**VirusTotal** (virustotal.com) — Bir faylı və ya URL-i **70+ antivirus motoru** ilə eyni anda skan edən platformadır.

**Necə işləyir:**

```
Sən -> Fayl/URL/Hash yukledin
         |
VirusTotal -> 70+ antivirus motoruna gonderir
         |
Nəticə -> "37/72 motor zərərli dedi"
```

**3 üsulla axtariş edə bilərsən:**

```
1. Fayl yukle          -> Birbaşa fayl gondər
2. URL yoxla          -> Şübhəli linki analiz et
3. Hash ilə axtarış   -> MD5/SHA1/SHA256 hash kodu ile tap
```

**Hash nədir?**

Hash — bir faylın **barmaq izi** kimidir. Faylın içindəkilərdən yaradılan unikal kod.

```
normal.exe   -> SHA256: a1b2c3d4e5f6...
malware.exe  -> SHA256: 2de70ca737c1f4602517c555ddd54165432cf231ffc0e21fb2e23b9dd14e7fb4

Eyni hash = eyni fayl
Fərqli hash = fərqli fayl (1 bit dəyişsə belə tam fərqli hash)
```

**Nə vaxt istifadə edilir:**

```
- Şübhəli e-mail əki gəldi -> Hash al -> VirusTotal-a yoxla
- Naməlum .exe tapildi   -> VirusTotal-a yukle
- Phishing URL şübhəsi   -> VirusTotal-da URL yoxla
```

---

### Have I Been Pwned (HIBP)

**HIBP** (haveibeenpwned.com) — E-mail ünvanının hansı məlumat sızıntısında (data breach) iştirak edib-etmədiyini yoxlayan platformadır.

```
Sən: ali@gmail.com
HIBP: "Bu e-mail 3 sızıntıda tapıldı:
       - LinkedIn (2012) - 117M istifadəci
       - Adobe (2013) - 153M istifadəci
       - Dropbox (2012) - 68M istifadəci"
```

**Kibertəhlükəsizlikdə niyə vacibdir:**

```
- Öz e-mailinin sızdırılıb-sızdırılmadığını öyrən
- Şirkətin işçilərinin şifrələrinin sızıb-sizmadığını yoxla
- Credential stuffing hücumlarını anlamaq ucun
```

---

## Task 5 - Zəifliklər ve Exploitlər

### CVE nədir?

**CVE** = **C**ommon **V**ulnerabilities and **E**xposures = Ümumi Zəifliklər ve İfşalar

CVE — məlum zəifliklərin **standart identifikasiya sistemidir.** Dünyada kəşf edilən hər zəiflik CVE ID alır.

```
Format: CVE-[İL]-[NÖMRƏ]

Nümunələr:
CVE-2014-0160  -> Heartbleed (OpenSSL-in məşhur zəifliyi)
CVE-2017-0144  -> EternalBlue (WannaCry-nin istifadə etdiyi)
CVE-2021-44228 -> Log4Shell (Log4j zəifliyi)
CVE-2024-3094  -> xz Utils backdoor
```

### CVE-2024-3094 — Xz Utils Backdoor

Bu CVE-nin xüsusi hekayəsi var:

```
xz utils — Linux sistemlərindəki sıxıştırma aləti

2024-cu ildə məlum oldu ki:
- "Jia Tan" adlı kimisə 2 il ərzində
  xz layihəsinə töhfə verdi
- Etibarı qazandıqdan sonra
  gizli backdoor (arxa qapı) yerləşdirdi
- Bu backdoor SSH serverinə icazəsiz giriş imkanı yaradırdı
- Bir developer şans əsərindən tapdi!

Bu hadisə supply chain attack (təchizat zənciri hücumu)
nümunəsidir.
```

### NVD nədir?

**NVD** = **N**ational **V**ulnerability **D**atabase = Milli Zəiflik Verilənlər Bazası

CVE-lərə əlavə məlumat əlavə edir:
- **CVSS skoru** (0-10 arası ciddilik dərəcəsi)
- Təsir etdiyi sistemlər
- Həll yolları (patch)

```
CVSS Skoru:
0.0      -> Heç bir risk
1.0-3.9  -> Aşağı
4.0-6.9  -> Orta
7.0-8.9  -> Yüksək
9.0-10.0 -> Kritik
```

---

### Exploit-DB nədir?

**Exploit-DB** (exploit-db.com) — Məlum zəiflikler ucun **sınaqdan keçirilmiş exploit kodlarının** açıq arxividir.

**Necə axtariş edilir:**

```
1. CVE ID ilə: CVE-2021-44228 yaz -> Log4Shell exploiti tap
2. Proqram adı ilə: "Apache 2.4.49" -> Bu versiyaya aid exploitlər
3. Platform ile: Windows, Linux, Web filtri
```

**Exploit növləri:**

```
Verified (Doğrulanmış): Test edilib, işləyir
Unverified:             Test edilməyib, risk var
```

**ÇOX VACIB XƏBƏRDARLIQ:**

```
Exploit-DB-dəki exploitləri yalnız:
  - İcazəli pentest muhitinde
  - Oz sisteminde (CTF, lab)
  - Hüquqi sözleşme olan testde

istifadə et!

İcazəsiz sistemdə exploit işlətmək
CINAYATDIR ve hüquqi nəticələri var.
```

---

### CVE, NVD ve Exploit-DB Arasindaki Fərq

```
CVE:        "Bu zəiflik mövcuddur, ID-si CVE-2021-44228-dir"
             (Sadəcə katalog, siyahı)

NVD:        "Bu zəifliyin CVSS skoru 10.0-dır,
              Log4j 2.0-2.14.1 versiyalarını əhatə edir"
             (Ətraflı texniki məlumat)

Exploit-DB: "Bu zəifliyi istismar etmek ucun
              kod budur: [exploit kodu]"
             (Praktiki istismar kodu)
```

---

## Task 6 - Texniki Senedler

### Niyə Rəsmi Sənəd?

İnternetdəki blog yazıları köhnə ola bilər, yanlış ola bilər. **Rəsmi texniki sənəd** — bir alətin ən etibarlı, ən dəqiq məlumat mənbəyidir.

---

### Linux Man Pages (Kisi Səhifələri)

**Man pages** — Linux/Unix sistemlərindəki hər əmrin daxili sənədidir.

```bash
# İstifadəsi:
man [əmr]

# Nümunələr:
man ls         -> ls əmrinin bütün parametrləri
man ip         -> ip əmrinin tam sənədi
man ssh        -> SSH-nin tam sənədi
man grep       -> grep-in bütün seçimləri
```

**Man page quruluşu:**

```
NAME        -> Əmrin adı və qısa izahı
SYNOPSIS    -> Sintaksis
DESCRIPTION -> Ətraflı izah
OPTIONS     -> Parametrlər (-a, -l, -h kimi)
EXAMPLES    -> Nümunələr
SEE ALSO    -> Əlaqəli əmrlər
```

**`cat` əmri nədir?**

`cat` = **cat**enate = birləşdirmək

```bash
# Əsl məqsədi — faylları birləşdirmək:
cat fayl1.txt fayl2.txt > birlesik.txt

# Amma ən çox istifadə olunan üsul — fayl oxumaq:
cat /etc/passwd
cat /var/log/syslog

# Adı "concatenate"-dən gəlir, amma hamı oxumaq ucun işlədir.
```

---

### Windows netstat -b parametri

Windows-da `netstat` əmri şəbəkə əlaqələrini göstərir. `-b` parametri **hər əlaqənin hansı proqrama aid olduğunu** göstərir.

```
C:\> netstat -b

Active Connections
  Proto  Local Address    Foreign Address    State
  TCP    0.0.0.0:80       0.0.0.0:0          LISTENING
 [httpd.exe]
  TCP    192.168.1.5:443  13.32.5.1:https    ESTABLISHED
 [chrome.exe]
```

> **Qeyd:** `-b` parametri **Administrator** səlahiyyəti tələb edir.

---

### Microsoft Docs

Windows alətləri, PowerShell, Azure, .NET sənədləri ucun:
**docs.microsoft.com** — rəsmi Microsoft sənəd platforması.

```
Axtariş nümunələri:
"netstat windows" -> Bütün parametrləri tap
"PowerShell Get-Process" -> Əmrin tam sintaksisi
"Windows Event ID 4625" -> Failed login event izahı
```

---

### Digər Vacib Sənəd Mənbələri

| Platforma | Nə ucun? |
|-----------|----------|
| **Linux man pages** | Bütün Linux əmrləri |
| **Microsoft Docs** | Windows, PowerShell, Azure |
| **Snort Docs** | IDS/IPS qaydaları |
| **NIST SP 800 seriası** | Kibertəhlükəsizlik standartları |
| **OWASP** | Veb tətbiq zəiflikləri |
| **RFC sənədlər** | İnternet protokollarının rəsmi tərifi |

---

## Task 7 - Sosial Media ve OSINT

### OSINT nədir?

**OSINT** = **O**pen **S**ource **I**ntelligence = Açıq Mənbə Kəşfiyyatı

OSINT — ictimai əlçatımlı məlumatlardan istifadə edərək bir şəxs, şirkət, və ya sistem haqqında məlumat toplama metodudur.

```
OSINT mənbəleri:
  - Sosial media profilləri
  - Şirkətin veb saytı
  - İş elanları
  - Açıq sənədlər (PDF, DOCX)
  - Domen qeydiyyat məlumatları (WHOIS)
  - DNS qeydlər
  - Şəkil metadata-sı (EXIF)
```

**Vacib:** OSINT **qanuni** məlumatlarla, **açıq** mənbələrdən aparılır. Sisteme izinsiz daxil olmaq OSINT deyil, cinayətdir.

---

### LinkedIn — Professional Kəşfiyyat

**LinkedIn** — işçilərin professional məlumatlarını tapmaq ucun ən güclü platformadır.

**Pentest ucun nə tapila bilər:**

```
Hədəf: XYZ Şirkəti

LinkedIn-də axtariş:
  - İşçilərin adları ve vəzifələri
  - İT departamenti kimlərdən ibarət?
  - Hansı texnologiyaları işlədirlər?
    (Profillərdə: "AWS, Docker, Kubernetes" yazır)
  - Yeni işə giriblər? (Sistemi hənuz bilmirlər?)
  - Kimi ixrac ediblər? (Narazı işçi?)
```

**Nümunə ssenari:**

```
LinkedIn-de IT Manager tapdin:
  - 10 ildir Microsoft Azure işlədir
  - Son layihəsi: "Active Directory migration"
  - Sertifikatları: CISSP, Azure Admin

Bu məlumat deyir:
  - Şirkətin bulud infrastrukturu Azure-da
  - AD sistemi yeni miqrasiya edir (keçid dövrü = zəif nöqtə?)
  - CISSP sertifikati = güclü security awareness
```

---

### Facebook — Personal Məlumat

**Facebook** — şəxsi həyat məlumatlarını tapma ucun istifadə olunur.

**Niyə vacibdir:**

```
Sosial mühəndislik (Social Engineering) ucun:
  - Uşaqlıq şəhəri -> Güvenlik sualının cavabı ola bilər
  - İlk məktəbi    -> Güvenlik sualının cavabı ola bilər
  - Heyvan adi     -> Güvenlik sualının cavabı ola bilər
  - Ailə üzvləri   -> Spear phishing ucun hədəf
  - Doğum günü     -> Bəzən PIN/şifrə kimi işlədilir
```

**Nümunə:**

```
"Şifrəmi Unutdum" -> "Uşaqlığınızda hansı şəhərdə yaşadınız?"

Hacker Facebook-da baxdı:
  Profile: "Doğma şəhərim Gəncədir! ❤️" -> Cavabı tapdı!
```

> **Etika qeydi:** OSINT qanuni məlumatla aparılmalıdır. Şəxsin məlumatini onun razılığı olmadan ictimaiyyətə açmaq etik deyil.

---

### Digər OSINT Mənbəleri

| Mənbə | Nə üçün istifadə olunur? |
|-------|--------------------------|
| **Twitter/X** | Real vaxtlı hadisələr, texniki insanların paylaşımları |
| **GitHub** | Kodda gizlənmiş API açarları, şifrələr, konfiq fayllar |
| **WHOIS** | Domenin kim tərəfindən qeydiyyata alındığı |
| **Shodan** | Şirkətin açıq internet cihazları |
| **Google Maps** | Şirkətin fiziki yeri, ətrafındakı binalar |
| **Wayback Machine** | Silinmiş veb səhifələrin köhnə versiyaları |

---

## Task 8 - Nəticə

### Öyrəndiklərinin Xülasəsi

```
Search Skills
|
+-- Mənbə Qiymətləndirmə
|     Mənbə, Dəlil, Obyektivlik, Təsdiq
|     Snake Oil -> Saxta kriptografiya
|
+-- Google Dork Operatorlari
|     "dırnaq"   -> Dəqiq ifadə
|     site:      -> Sayt filtri
|     -           -> İstisna et
|     filetype:  -> Fayl növü
|
+-- Xüsusi Axtarış Motorları
|     Shodan     -> İnternetə bağlı cihazlar
|     Censys     -> Domenler, sertifikatlar, portlar
|     VirusTotal -> Fayl/URL/Hash analizi
|     HIBP       -> Email sızıntısı yoxlama
|
+-- Zəiflik Verilənlər Bazaları
|     CVE        -> Zəiflik identifikatoru (CVE-YYYY-NNNNN)
|     NVD        -> CVSS skoru + ətraflı məlumat
|     Exploit-DB -> İstismar kodları (yalnız icazəli testde!)
|
+-- Texniki Sənədlər
|     man [əmr]  -> Linux sənədi
|     Microsoft Docs -> Windows sənədi
|     cat = concatenate, ss = socket statistics
|
+-- OSINT
      LinkedIn -> Professional məlumat
      Facebook -> Şəxsi məlumat
      GitHub   -> Gizlənmiş açarlar, konfiqler
```

---

### Real Dünyada Bu Bacarıqlar Necə Birləşir?

**Ssenariya:** Şirkətə pentest aparirsan.

```
1. LinkedIn-de işçiləri tap (OSINT)
        |
2. Shodan-da şirkətin açıq portlarını tap (Xüsusi axtarış)
        |
3. Tapılan servis versiyasını CVE-də yoxla (Zəiflik DB)
        |
4. Exploit-DB-dəki exploit-i tap (İstismar kodu)
        |
5. Man page-dən alət sintaksisini öyrən (Texniki sənəd)
        |
6. Nəticəni düzgün mənbədən doğrula (Mənbə qiymətləndirməsi)
```

Bütün bu addımlar **axtariş bacarığına** əsaslanır.

---

*Mənbə: TryHackMe - Search Skills - https://tryhackme.com/room/searchskills*
