
XSS zafiyeti

Cross-Site Scripting (XSS), kullanıcıdan gelen verinin uygun şekilde encode edilmeden veya filtrelenmeden, başka bir kullanıcının tarayıcısında kod olarak yorumlanmasına neden olan bir güvenlik açığıdır.

Bu olayın başlangıcını anlamak için request–response döngüsüne bakmak gerekir.

Bir kullanıcı web uygulamasına bir HTTP request gönderir. Sunucu bu isteğe karşılık bir HTTP response üretir. Response temel olarak iki ana bölümden oluşur: Header, Body

XSS zafiyeti genellikle response’un body kısmında ortaya çıkar. Çünkü body içinde HTML, JavaScript, CSS yer alır ve tarayıcı bu içeriği parse ederek DOM oluşturur.

Eğer kullanıcı girdisi, encode edilmeden response body içine yerleştirilirse, tarayıcı bu girdiyi veri olarak değil kod olarak yorumlayabilir.


Bu noktada kritik olan şey şudur :

Sunucu açısından response body sadece bir metin çıktısıdır ama tarayıcı açısından bu çıktı, yorumlanacak bir programdır.
Tarayıcı response body’yi aldıktan sonra şu adımlar gerçekleşir:

1- browser HTML içeriği parser etmeye çalışır ve gelen içeriği tokenize eder.
2- Token’lardan bir DOM  ağacı oluşturulur.
3- Eğer <script> tag’i, inline event handler (onclick gibi) veya JavaScript context’i oluşursa, JavaScript engine devreye girer.
4- Script, sayfanın origin’i bağlamında çalıştırılır.

XSS’in ortaya çıktığı asıl an, HTML parser’ın kullanıcı girdisini veri olarak değil kod olarak algıladığı andır.


peki xss türleri nelerdir ? 

Reflected XSS

Reflected XSS, kullanıcıdan gelen verinin sunucu tarafından response içinde anlık olarak geri yansıtılması sonucu oluşur.
Veri akışı şu şekildedir:
Kullanıcı → HTTP Request → Sunucu → HTTP Response → Browser

Bu senaryoda:

Kullanıcı URL parametresi veya form verisi gönderir. Sunucu bu veriyi encode etmeden response body içine yerleştirir. Tarayıcı bu çıktıyı parse eder ve kod olarak yorumlar.

Önemli nokta:

Bu tür XSS genellikle kalıcı değildir. Payload sunucuda saklanmaz.
Saldırgan, mağduru özel hazırlanmış bir linke tıklamaya ikna etmek zorundadır.

Reflected XSS’in temel problemi ---> Output encoding eksikliği + anlık veri yansıtımıdır.


Stored XSS

Stored XSS’te kullanıcı girdisi sunucu tarafında kalıcı olarak saklanır.

Veri akışı şu şekildedir:
Saldırgan → Sunucu (veri kaydedilir) → Veritabanı → Başka kullanıcı → Response → Browser

Bu senaryoda:

Saldırgan payload’u bir yorum alanına, profil alanına vb. kaydeder. Sunucu bu veriyi saklar. Başka bir kullanıcı ilgili sayfayı ziyaret ettiğinde payload response içinde yer alır. Tarayıcı bunu parse eder ve çalıştırır. Stored XSS, Reflected XSS’e göre daha tehlikelidir çünkü Sosyal mühendislik gerektirmeyebilir. Otomatik olarak birden fazla kullanıcıyı etkileyebilir.

Buradaki root cause yine aynıdır:

Untrusted input’un encode edilmeden output’a dahil edilmesi.

DOM XSS

DOM-Based XSS’te zafiyet tamamen client-side JavaScript kodundan kaynaklanır.

Bu durumda: Sunucu güvenli response üretmiş olabilir. Ama tarayıcıdaki JavaScript kodu, kullanıcıdan gelen veriyi unsafe şekilde DOM’a yazabilir.

Örnek veri akışı:

Kullanıcı URL parametresi → Browser → JavaScript → innerHTML → DOM → Execution

Burada dikkat edilmesi gereken kavramlar:

Source: Kullanıcıdan gelen veri (location.search, location.hash vb.)

Sink: Bu verinin yazıldığı tehlikeli fonksiyonlar (innerHTML, document.write, eval vb.)

DOM XSS’in ayırt edici özelliği şudur:

Zafiyet server response’unda görünmeyebilir.
Exploit tamamen tarayıcı içinde gerçekleşir.
