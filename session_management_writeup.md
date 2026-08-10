# Session Management — Öyrənmə Writeup'u
> TryHackMe: Session Management Room

---

## 1. Session Management Nədir?

HTTP protokolu **stateless**-dir — yəni hər request bir-birindən xəbərsizdir. Server hər dəfə "bu kim?" deyə soruşur. Session management bu problemi həll edir: istifadəçiyə unikal ID verilir və hər requestdə bu ID göndərilir.

```
İstifadəçi login olur
        ↓
Server unikal session ID verir
        ↓
Hər requestdə bu ID göndərilir
        ↓
Server "bu kimdir?" sualını cavablayır
```

---

## 2. Session Management Lifecycle — 4 Mərhələ

### 2.1 Session Creation (Yaradılma)
Login olduqdan sonra server sənə unikal ID verir. Bəzi saytlar login olmadan əvvəl də session yaradır.

**Potensial zəifliklər:**
- Zəif/təxmin edilən session ID-lər
- JWT imzasının yoxlanmaması
- Session Fixation (login sonra ID dəyişmirsə)
- Təhlükəsiz olmayan redirect vasitəsilə token oğurlanması

### 2.2 Session Tracking (İzlənmə)
Hər requestdə session ID göndərilir, server kimin nəyə icazəsi olduğunu yoxlayır.

**Potensial zəifliklər:**
- **Vertical Bypass** — Normal istifadəçi admin funksiyalarına çata bilir
- **Horizontal Bypass** — İstifadəçi başqasının datasına çata bilir
- Kifayətsiz loglama — Hücum zamanı nə olduğunu anlamaq olmur

### 2.3 Session Expiry (Müddəti)
Session-un ömrü bitəndə yenidən login lazımdır.

**Potensial zəifliklər:**
- Çox uzun session ömrü — oğurlanmış session uzun müddət işləyir
- Lokasiya dəyişirsə session bitməlidir

### 2.4 Session Termination (Bitirilmə)
Logout edildikdə session həm client, həm də server tərəfindən silinməlidir.

**Potensial zəifliklər:**
- Yalnız client-side silinmə — server hələ qəbul edir
- JWT blocklist-i olmadan logout mümkün deyil

---

## 3. IAAA Modeli

| Mərhələ | Nədir | Session ilə Əlaqəsi |
|---|---|---|
| **Identification** | "Mən kiməm" iddiası (username) | — |
| **Authentication** | İddia sübut edilir (şifrə) | Session yaradılır |
| **Authorisation** | İcazələr yoxlanır | Session tracking |
| **Accountability** | Hərəkətlər loglanır | Session loglanır |

---

## 4. Cookie vs Token

### Cookie-Based Session
```
Server → Brauzer: Set-Cookie: session=abc123
Brauzer: Avtomatik hər requestdə göndərir
```

**Mühüm atributlar:**
| Atribut | Məqsəd |
|---|---|
| `Secure` | Yalnız HTTPS ilə göndərilir |
| `HTTPOnly` | JavaScript oxuya bilməz (XSS qoruması) |
| `Expire` | Session nə vaxt bitir |
| `SameSite` | CSRF hücumlarından qoruyur |

### Token-Based Session (JWT)
```
Server → Brauzer: Token (request body-də)
JavaScript: LocalStorage-də saxlayır
Hər requestdə: Authorization: Bearer <token>
```

**Müqayisə:**
```
                  COOKİE         TOKEN
Avtomatik?        ✅ Brauzer     ❌ JS lazım
CSRF təhlükəsi?   ❌ Var         ✅ Yoxdur
Fərqli domenlər?  ❌ Çətin       ✅ Asan
```

---

## 5. Lab — Praktik Təhlil

### 5.1 Enumeration (Kəşfiyyat)

**Addım 1: Sayta ilk girişdə nə olur?**
- Cookie var mı? → Unauthenticated session izlənirmi?
- Developer Tools → Network → Cookies yoxla

**Addım 2: Qeydiyyat**
- Student → Açıq qeydiyyat
- Lecturer → Verification code lazım (privilege ayrımı var)

**Addım 3: Login olduqdan sonra**
- `Set-Cookie` header-i gəlirmi?
- `HTTPOnly` flag varmı?
- Cookie dəyəri nə qədər randomdur?

**Addım 4: Hər requestdə**
- Cookie göndərilirmi?
- Cookie olmadan nə olur?

**Addım 5: Logout**
- Cookie client-side silinirmi?
- Köhnə cookie ilə request göndər → Server qəbul edirmi?

---

### 5.2 Local Storage Analizi

```
F12 → Application → Local Storage
```

Burada saxlanan dəyərlər:
```json
{
  "userRole": "student",
  "id": 2,
  "username": "test"
}
```

**Test et:**
```
userRole: "student" → "lecturer"  dəyişdir → Refresh
id: 2 → 1  dəyişdir → Refresh
```

Əgər server bu dəyərləri yoxlamırsa → **Authorisation Bypass!**

---

### 5.3 Aşkar Edilən Zəifliklər

| Zəiflik | Mərhələ | İzah |
|---|---|---|
| Çox uzun session ömrü | Session Expiry | Session çox gec bitir |
| Client-side token manipulyasiyası | Session Tracking | userRole dəyişdirilə bilir |
| Server-side yoxlama yoxdur | Session Tracking | Token dəyərləri server tərəfindən doğrulanmır |
| 500 error invalid session-da | Session Termination | Düzgün xəta idarəsi yoxdur |

---

## 6. Öyrənilənlər

### Nəyi Axtarmalıyıq?

```
1. Cookie atributlarını yoxla
   → Secure, HTTPOnly, SameSite varmı?

2. Session ID-nin gücünü yoxla
   → Təxmin etmək olarmı?

3. Local Storage-i yoxla
   → Həssas məlumat saxlanırmi?

4. Rol dəyərlərini manipulyasiya et
   → Server yoxlayırmı?

5. Logout-dan sonra köhnə session-u test et
   → Server-side terminate olunurmu?

6. Cookie olmadan requestlər göndər
   → Nəyə çatmaq olur?
```

### Düzgün Tətbiq Necə Olmalıdır?

```
✅ Session ID → kriptoqrafik random
✅ Login sonra → yeni session ID (fixation qoruması)
✅ HTTPOnly + Secure + SameSite atributları
✅ Qısa session ömrü (kontekstə görə)
✅ Logout → server-side terminate
✅ Bütün rol yoxlamaları server-side
✅ Hər əməliyyat loglanır
```

---

## 7. Faydalı Əmrlər

```bash
# Cookie-ni manual göndər
curl -s -b "session=abc123" http://TARGET_IP/api/data

# Köhnə session-u test et
curl -s -b "session=KÖHNƏ_DƏYƏR" http://TARGET_IP/dashboard

# Bütün response header-lərini gör
curl -s -I http://TARGET_IP/login
```

---

## 8. Xülasə Cədvəli

| Mərhələ | Axtarılan Zəiflik | Test Metodu |
|---|---|---|
| Session Creation | Zəif ID, Fixation | ID dəyərini analiz et |
| Session Tracking | Bypass, loglama | Rol dəyişdir, başqa ID-ləri test et |
| Session Expiry | Uzun ömür | Müddəti yoxla |
| Session Termination | Server-side silinmir | Logout sonra köhnə ID ilə test et |
