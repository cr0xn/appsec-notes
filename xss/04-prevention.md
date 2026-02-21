# XSS Prevention (Savunma Teknikleri)

XSS savunmasının temel felsefesi, tarayıcı parser'ının **veri (data)** ile **kodu (code)** birbirine karıştırmasını engellemektir. Savunma, sadece zararlı karakterleri temizlemek değil, verinin gireceği "Context"e (Bağlam) göre onu zararsız bir metin yığınına dönüştürmektir.

## 1. Temel Strateji: Output Encoding (Çıkış Kodlama)
XSS zafiyetinin birincil çözümü, verinin ekrana yazılmadan hemen önce kodlanmasıdır. Bu işleme "Context-Aware Output Encoding" denir.

### Neden Input Validation (Giriş Denetimi) Yeterli Değildir?
- Giriş denetimi "kara liste" (black-list) mantığına dayanır ve atlatılması kolaydır.
- Veri bir yerde güvenli görünürken (örn: veritabanı), başka bir context'te (örn: bir JS değişkeni içinde) tehlikeli olabilir.
- **Kural:** Veri girişte doğrulanmalı (Validation), çıkışta ise mutlaka kodlanmalıdır (Encoding).



## 2. Bağlama Duyarlı Koruma Tablosu

| Context | Örnek Yerleşim | Savunma Yöntemi |
| :--- | :--- | :--- |
| **HTML Body** | `<div>...</div>` | HTML Entity Encoding (`<` -> `&lt;`) |
| **Attribute** | `<input value="...">` | Attribute Encoding (Tırnaklar encode edilir) |
| **JavaScript** | `var x = '...';` | Unicode/Hex Escaping (`'` -> `\x27`) |
| **URL** | `<a href="...">` | URL Encoding + Protocol Allowlist |

### A. HTML Body İçin Savunma
Veri HTML etiketleri arasına düşüyorsa, parser'ın yeni bir tag başlatmasını engellemek için şu 5 temel karakter encode edilmelidir:
- `<`  → `&lt;`
- `>`  → `&gt;`
- `&`  → `&amp;`
- `"`  → `&quot;`
- `'`  → `&#x27;`



### B. Attribute Context Savunması
Girdi bir attribute değeri içindeyse (`<input value="... ">`), savunma stratejisi şudur:
- **Kural:** Attribute değerlerini her zaman çift tırnak (`" "`) içinde tutun.
- **Encoding:** Sadece `"` karakterini `&quot;` yapmak yeterli görünse de, boşluk (space), `%`, `*` gibi karakterlerin de encode edilmesi (Non-Alphanumeric Encoding) tavsiye edilir. Çünkü bazı durumlarda parser tırnak olmadan da attribute bitişi algılayabilir.



### C. JavaScript Context Savunması
Senin notlarında belirttiğin gibi; burada HTML encoding (`&lt;`) işe yaramaz, çünkü JS engine bunu anlamaz.
- **Kural:** Veriyi asla doğrudan bir script bloğuna basmayın.
- **Teknik:** Eğer basılacaksa, Unicode Escaping kullanılmalıdır. 
  - `'` (tek tırnak) -> `\x27` veya `\u0027`
  - `"` (çift tırnak) -> `\x22` veya `\u0022`
- **Modern Yöntem:** Veriyi HTML içinde bir `data-*` attribute'una (HTML encode ederek) koyup, JS ile oradan çekmek en güvenli yoldur.



## 3. İkinci Savunma Hattı (Defense in Depth)

Kodlama (Encoding) atlandığında veya hatalı yapıldığında saldırıyı durduran mekanizmalardır.

### A. Content Security Policy (CSP)
Tarayıcıya hangi kaynaklardan script çalıştırabileceğini söyleyen bir HTTP Header'ıdır.
- **Örnek:** `Content-Security-Policy: script-src 'self';`
- **Etki:** Başka bir sunucudan (attacker.com) script çekilmesini ve inline (satır içi) `<script>` bloklarının çalışmasını engeller.

### B. HttpOnly Cookie Flag
XSS saldırısının en büyük amacı olan "Session Hijacking"i (oturum çalma) engeller.
- **Örnek:** `Set-Cookie: session=abc123; HttpOnly`
- **Etki:** JavaScript'in `document.cookie` ile bu veriye erişmesini engeller. Saldırgan XSS tetiklese bile cookie'yi çalamaz.

### C. X-Content-Type-Options: nosniff
Tarayıcının bir metin dosyasını (örneğin bir .txt dosyasını) JavaScript olarak "tahmin edip" (MIME sniffing) çalıştırmasını engeller.




## 4. Güvenli DOM Kullanımı (DOM XSS Önleme)

DOM tabanlı XSS'i engellemek için "Tehlikeli Sink"lerden kaçınmak gerekir.

- **Kötü:** `element.innerHTML = USER_INPUT;` (Veriyi HTML olarak yorumlar)
- **İyi:** `element.textContent = USER_INPUT;` (Veriyi sadece düz metin olarak işler)
- **Kötü:** `eval()`, `setTimeout(string)`, `document.write()`
- **İyi:** `JSON.parse()` veya güvenli DOM manipülasyon metotları.



































































