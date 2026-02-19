Context Nedir?

Bir context, kullanıcı girdisinin tarayıcı tarafından hangi kurallarla parse edileceğini belirleyen yürütme ortamıdır. Tarayıcı tek bir yorumlayıcı değildir. HTML, attribute, JavaScript ve URL gibi farklı parsing modlarına sahiptir. Bir girdinin güvenli olup olmaması, bulunduğu context’e bağlıdır.

XSS, kullanıcı girdisinin bulunduğu context’e uygun şekilde encode edilmemesi sonucu ortaya çıkar.


1- HTML Context

HTML context, kullanıcı girdisinin bir elementin body kısmı içinde, HTML parser tarafından Data State modunda parse edilmesidir.

Örnek: 

<div>USER_INPUT</div>

Kullanıcı girdisinin test" olduğunu varsayalım:

<div>test"</div>

Bu durumda çift tırnak karakteri HTML parser tarafından özel bir kontrol karakteri olarak değerlendirilmez. Çünkü parser şu anda Data State içindedir.

Data State’te:

< karakteri yeni bir tag başlatabilir 
& karakteri entity başlatabilir
Diğer karakterler (`, ", harfler vb.) düz metin olarak DOM’a eklenir. Dolayısıyla ", DOM’a bir text node olarak eklenir ve markup yapısını bozmaz.

HTML context’te execution oluşabilmesi için Parser’ın Data State’ten çıkması gerekir. Bu geçiş genellikle < karakteri ile tetiklenir.Yani risk, belirli karakterlerin parser state’ini değiştirebilme kapasitesinden kaynaklanır. Ancak parser state geçişine neden olabilecek karakterler (özellikle <) encode edilmediğinde, veri markup’a dönüşebilir ve execution zinciri başlatılabilir.


2- Attribute Context

Attribute context, kullanıcı girdisinin bir HTML elementinin attribute değeri içinde parse edilmesidir.

Örnek:

```
<input value="USER_INPUT">
```

Kullanıcı girdisinin test" olduğunu varsayalım:

```
<input value="test"">
```

Bu durumda HTML parser artık Data State’te değildir.
Parser şu anda Attribute Value (Double-Quoted) State içindedir.

Bu state’te ' " ' karakteri attribute değerini sonlandırır, ' & ' entity başlatabilir, ' < '  bazı durumlarda parsing davranışını etkileyebilir
Diğer karakterler attribute değerine eklenir

Parser şunu okur:

value="test"

İlk " karakteri attribute değerini kapatır. Sonraki " artık yeni bir attribute başlangıcı gibi yorumlanabilir.

Yani burada olan şey veri artık sadece metin değildir, Attribute boundary kırılmıştır, Parser state değişmiştir. Bu, HTML body context’ten temel farktır.

Attribute context’te kritik olan şey: String boundary kontrolüdür. Çift tırnak (") veya tek tırnak (') kullanılan attribute’larda, bu karakterler parser için kontrol karakteridir.

Body context’te ", düz metinken, Attribute context’te ", boundary sonlandırıcıdır. Bu yüzden aynı input farklı context’te farklı güvenlik sonucu doğurur.

Attribute boundary kırıldığında, Yeni attribute eklenebilir, Event handler attribute oluşturulabilir, URL veya başka execution tetikleyici attribute’lar eklenebilir Burada execution doğrudan gerçekleşmez. Ancak parser’a yeni davranış alanı açılır. XSS genellikle bu boundary kırılmasından sonra başlar.



3- JavaScript Context

JavaScript context, kullanıcı girdisinin bir <script> bloğu içinde veya inline JavaScript ifadesi içinde parse edilmesidir.

Örnek:

```
<script>
var name = "USER_INPUT";
</script>
```

Kullanıcı girdisinin test" olduğunu varsayalım:

```
<script>
var name = "test"";
</script>
```

Bu noktada HTML parser <script> tag’ine girdiği anda parsing modunu değiştirir. Artık içerik HTML Data State’te değil, Script Data State içindedir.
Daha sonra içerik JavaScript engine’e aktarılır.

Parsing Katmanları

JavaScript context’te iki aşama vardır:

HTML parser <script> bloğunu tanımlar.

İçerik JavaScript engine tarafından parse edilir.

Yani burada iki ayrı yorumlama katmanı vardır.

XSS genellikle ikinci katmanda, yani JavaScript parsing aşamasında oluşur.

String Boundary Problemi

Örnekte:

var name = "test"";


JavaScript engine şunu görür:

var name = "test"


İlk " string’i sonlandırır.

Sonraki " artık yeni bir token başlangıcıdır.
Bu noktada:

Yeni bir ifade parse edilebilir, Syntax değişebilir, Execution zinciri oluşabilir. Buradaki kritik kavram: String boundary kırılması.


HTML encoding yalnızca <, >, & gibi karakterleri etkiler.

Ancak JavaScript string context’te problem ---> " ,  ' \ ; )  newline karakterleri gibi JavaScript’e özgü boundary karakterleridir. Bu nedenle:
HTML için güvenli olan veri, javaScript context’te güvensiz olabilir.

Context Değişiminin Önemi

HTML context’te: ---> Risk parser state değişimi ile oluşur.
JavaScript context’te: ---> Risk lexical parsing ile oluşur.
Yani burada artık markup değil, dil kuralları devrededir.

JavaScript context’te execution için: ---> String kapanmalı, Yeni statement parse edilebilmeli, Engine bunu geçerli syntax olarak kabul etmeli
XSS burada, JavaScript engine’in veri ile kodu ayırt edememesi sonucu ortaya çıkar.





/////////////////////////////////////////////////////////////////////////////////////////////////////////////








URL Context

URL context, kullanıcı girdisinin bir URL değerinin parçası olarak parse edilmesidir.

Örnek:


<a href="USER_INPUT">Link</a>


Bu durumda parser ilk aşamada HTML Attribute Value State içindedir.
Ancak kullanıcı linke tıkladığında ikinci bir parsing süreci başlar:

URL parsing ve navigation süreci

Yani burada iki aşama vardır:

HTML parser attribute değerini okur

Tarayıcı URL’yi çözümleyerek (resolve ederek) işlem yapar

Çift Katmanlı Yorumlama

HTML parser için:

href="example"


sadece bir string değerdir.

Ancak tarayıcı için bu değer:

Bir scheme içerir mi?

Aynı origin mi?

Relative mi?

Special protocol mü?

gibi kontrollerden geçer.

Bu noktada risk HTML parsing’den değil, URL interpretation’dan doğar.

Scheme Kavramı

Bir URL şu formattadır:

scheme://host/path


Tarayıcı scheme’e göre davranır.

Örneğin:

http / https → navigation

mailto → mail client

data → inline data

Diğer özel protokoller → farklı execution yolları

Yani burada risk:

Kullanıcı girdisinin URL semantic anlamını değiştirmesidir.

HTML Context ile Farkı

HTML context’te risk:

Parser state değişimi

URL context’te risk:

Semantic interpretation değişimi

Yani burada boundary kırılması değil, anlam değişimi vardır.

Önemli Nokta

URL context’te güvenlik yalnızca HTML encoding ile sağlanamaz.

Çünkü problem markup değil, URL’nin davranış modelidir.

Bu nedenle burada gerekli olan:

Scheme validation

Allowlist yaklaşımı

URL normalizasyonu

Sonuç

URL context, kullanıcı girdisinin bir URL değeri olarak yorumlandığı durumdur.

Risk, HTML parser’dan değil, tarayıcının URL çözümleme ve navigation davranışından kaynaklanır.






DOM Context (Client-Side Execution)

Burada artık server output değil, client-side data flow konuşacağız.

DOM context, kullanıcı girdisinin server response’undan bağımsız olarak, tarayıcı içindeki JavaScript kodu tarafından DOM’a yazılması durumudur.

Bu senaryoda:

Server güvenli HTML üretmiş olabilir.

Ancak client-side JavaScript, kullanıcıdan gelen veriyi unsafe şekilde DOM’a yerleştirebilir.

Bu nedenle DOM-based XSS, klasik reflected veya stored modellerden farklıdır.

Veri Akışı Modeli

DOM context’te temel model şudur:

Source → JavaScript → Sink → DOM → Execution


Burada kritik kavramlar:

Source

Kullanıcıdan gelen veya kontrol edilebilen veridir.

Örnekler:

location.search

location.hash

document.referrer

window.name

postMessage verisi

Sink

Bu verinin yazıldığı tehlikeli DOM API’leridir.

Örnekler:

innerHTML

outerHTML

insertAdjacentHTML

document.write

eval

setTimeout(string)

Risk, verinin bu sink’lere doğrudan aktarılmasıyla oluşur.

Server’dan Farkı

Reflected ve Stored XSS’te:

Payload response body içinde görünür.

DOM-based XSS’te:

Payload response’ta görünmeyebilir.

Execution tamamen browser içinde gerçekleşir.

Yani burada trust boundary:

JavaScript runtime içindedir.

Parser Perspektifi

DOM context’te iki ayrı parsing olabilir:

JavaScript engine veriyi işler.

innerHTML gibi bir sink kullanıldığında, veri tekrar HTML parser’a gönderilir.

Bu, ikinci bir parsing döngüsü oluşturur.

Yani veri:

Önce string olarak değerlendirilir

Sonra markup olarak yeniden parse edilir

Bu çift yorumlama XSS’e zemin hazırlar.

Neden Daha Zor Tespit Edilir?

Çünkü:

Server response temiz görünebilir.

Güvenlik testleri yalnızca HTML çıktıya bakıyorsa zafiyet görünmeyebilir.

Exploit runtime sırasında oluşur.

Bu nedenle DOM-based XSS, static output analiziyle her zaman yakalanamaz.

Önemli Nokta

DOM context’te problem genellikle encoding eksikliğinden değil:

Güvensiz DOM API kullanımından kaynaklanır.

Bu nedenle çözüm:

innerHTML yerine textContent kullanmak

eval yerine güvenli alternatifler kullanmak

Veri akışını source–sink modeliyle analiz etmek

Diğer Context’lerle Karşılaştırma
Context	Risk Noktası	Parsing Katmanı
HTML Body	Tag boundary	HTML parser
Attribute	Attribute boundary	HTML parser
JavaScript	String boundary	JS engine
URL	Scheme interpretation	URL resolver
DOM	Source → Sink akışı	JS + HTML parser
Sonuç

DOM context, kullanıcı girdisinin client-side JavaScript tarafından DOM’a yazılması sonucu execution oluştuğu durumdur.

Burada zafiyet:

Server response’ta değil

Browser içindeki veri akışında

ortaya çıkar.












Final Summary: Context-Aware Security Modeli

XSS, tek tip bir zafiyet değildir.
Farklı execution context’lerde ortaya çıkan, ancak aynı kök probleme dayanan bir güvenlik ihlalidir.

Bu kök problem şudur:

Kullanıcı girdisinin bulunduğu parsing context’e uygun şekilde işlenmemesi.

Tarayıcı tek bir yorumlayıcı değildir. İçinde birden fazla parsing katmanı vardır:

HTML parser

Attribute parsing mekanizması

JavaScript engine

URL resolver

DOM API’leri

Her biri farklı kurallarla çalışır.
Bir context’te güvenli olan veri, başka bir context’te executable olabilir.

Context’lerin Ortak Noktası

Tüm context’lerde zafiyet şu noktada oluşur:

Veri ile kod arasındaki sınır (data–code boundary) ihlal edilir.

Parser veya engine, kullanıcı girdisini veri olarak değil, yürütülebilir yapı olarak yorumlar.

Farklılık şuradadır:

Context	Boundary Türü	Risk Mekanizması
HTML Body	Tag boundary	Parser state değişimi
Attribute	Attribute boundary	String sonlandırma
JavaScript	Expression boundary	Lexical parsing
URL	Semantic boundary	Scheme yorumlama
DOM	Source–Sink boundary	Runtime DOM injection
Neden Global Sanitize Yeterli Değildir?

Tek bir “sanitize” fonksiyonu XSS’i çözemez. Çünkü:

HTML context HTML encoding ister.

Attribute context quote-aware encoding ister.

JavaScript context string-aware encoding ister.

URL context scheme validation ister.

DOM context güvenli API kullanımı ister.

Yani güvenlik:

Input’u filtrelemekten değil, output’u bulunduğu context’e göre işlemekten geçer.

Bu yaklaşım “context-aware output encoding” olarak adlandırılır.

Mimari Perspektif

XSS aslında bir karakter problemi değildir.
Bir mimari tasarım problemidir.

Zafiyet, uygulamanın:

Hangi verinin güvenilir olduğu

Bu verinin hangi context’e gireceği

O context’in hangi parsing kurallarıyla çalıştığı

net şekilde modellenmediğinde ortaya çıkar.

Sonuç

XSS’i anlamak, payload ezberlemek değildir.
Parser state’leri, execution context’leri ve data–code boundary’leri anlamaktır.

Tüm XSS türleri, farklı yerlerde gerçekleşse de aynı ilkeye dayanır:

Untrusted input, uygun context kontrolü olmadan execution ortamına dahil edilmemelidir.
