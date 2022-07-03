## Due Jul 14, 2022, 23:59

# BANKING SYSTEM
Arkadaşlar merhaba, bu ödevimizde Spring Boot webservices(REST), Dosya İşlemleri ve Apache Kafka kullanarak basit bir bankacılık uygulaması geliştireceğiz.

Ödevimizde hesap yaratma, hesaba para yatırma, para transferi yapma, yapılan işlemlere dair loglama yapmak gibi işlemlerimiz olacaktır. 

Yapılan işlemlerin loglanmasını **KAFKA** aracılığıyla yapacağız.

Webservis istek ve cevapları tamamen **JSON** üzerinden yapılacaktır. **REST'in şartlarına (HTTP methods, HTTP status codes, URI structure, format)** uyulması bekliyorum.

İşlemlere tek tek bakacak olursak

### Webservice 1 ###

Birinci webservisimiz hesap yaratma üstüne olacaktır. Bir hesap yaratılırken kullanıcıdan isim, soyisim, email adresi, tc kimlik numarası ve hesap türü bilgisini alacağız. Yani servisimize şu şekilde bir istek gelecektir.
```json
    {
        "name" : "Abdullah",
        "surname" : "Yavuz",
        "email" : "bootcamp@bootcamp.com",
        "tc" : "12121212121",
        "type" : "TL"
    }
```

Buradak hesap türü üç farklı şekilde olabilir.
* Dolar
* TL
* Altın (Gram cinsinden birimlenecek)

istek bize geldikten sonra sayısal ve 10 basamaktan oluşan random bir hesap numarası oluşturacağız. Ve bir dosyaya(accounts.txt mesela) veya dosya ismini accountNumber şeklinde belirleyerek her bir accountu kendine özer bir dosyaya tüm detaylarıyla kaydedeceğiz.

Accountların tüm detayları demek istekten aldığımız değerler, bizim ürettiğimiz accountNumber, account balance'i(bakiye) ve accountta son güncellemenin ne zaman yapıldığına dair tutuğumuz zaman demek oluyor. **Accountlar her güncellendiğinde son güncelleme tarihih bilgisi o an olarak güncellemenizi istiyorum.**

Bunları dosyaya; geçen ödevdeki gibi virgüllerle veya boşluklarla ayırarak satır satır kaydedebilirsiniz.

```
	accountNumber,name,surname,email,tc,type,balance,lastUpdateDate
```
Hesabın bakiyesi(balance) sıfırdan başlayacaktır.

#### ÖNEMLI NOKTALAR ####
* Servisimize gelen dataların doğruluğundan emin olmalıyız arkadaşlar, type kısmında belirttiğim üç tanesinden farklı bir type gelirse kullanıcıya hata mesajı dönmenizi istiyorum. Hata mesajı dönerken uygun HTTP status kodunu kullanmanızı bekliyorum.(4xx'lerden birisi)
```json
{
    "message" : "Invalid Account Type: " + "inputtan gelen account type"
}
```

Spring boot tarafından bakarsak controller içindeki actionumuzda
```
        @RequestMapping(path = "size bırakıyorum", method = "size bırakıyorum")
	public ResponseEntity<> createAccount(@RequestBody AccountCreateRequest request){
		
	}
```
şeklinde bir methodumuz olacak. Hatırlayın **JSON** gelen istekleri okumak için isteğe özel bir sınıf yaratıp istekteki parametrelere karşılık sınıfta propertyler oluşturuyorduk. **@RequestBody** annotationu ile de gelen **JSON** isteğin objeye maplenmesini sağlıyorduk.

**AccountCreateRequest** sınıfımız buna göre şu şekilde olacaktır.
```
    public class AccountCreateRequest {
    	
    	private String name;
    	private String surname;
    	private String email;
    	private String tc;
    	private String type;
    	
    	//default constructor, getters and setters
    }
```
İşlemi eğer başarılı şekilde yerine getirebilirsek kullanıcıya şöyle bir mesaj dönmenizi istiyorum.

```json
{
    "message" : "Account Created",
    "accountNumber" : "random urettiğiniz numara"
}
```

Önemli diğer bir noktada cevaplara da cevap yapısına uygun sınıflar geliştirebileceğimiz arkadaşlar. Yani bu cevabı verebilmek için **AccountCreateSuccessResponse** diye bir response oluşturabiliriz.
```
    public class AccountCreateSuccessResponse {
    	
    	private String message;
    	private int accountNumber;
    	
    	//getters setter, default constructor
    }
```

ve return kısmında ResponseEntity içinde bunu dönebiliriz. Burada HTTP status kodunun uygun şekilde kullanılmasını bekliyorum. (2xx'lerden birisi).

Bu bir create işlemidir lütfen HTTP methodunuzu buna göre seçin.

### Webservice 2 ###
Bu servis direk olarak accountun detayını getiren servis olacaktır arkadaşlar. AccountNumber URL'den path olarak alınacaktır.

#### Önemli nokta ####
Arkadaşlar burada accountları accounts.txt'ye kaydederken(veya her accounta özel dosyaya) sizden birde her account için son güncelleme tarihi gibi bir bilgiyi de kaydetmenizi istiyorum. Bu bilgi account yaratılırken yaratılma anıdır, güncelleme oldugunda(mesela transfer veya para yatırma) bu süre güncellemenin yapıldıgı an olacaktır. Ve detay cevabında LAST MODIFIED headerina bu değeri vermenizi istiyorum.

Hatırlayın ResponseEntity sınıfı şu şekilde bu headeri set edebiliyordu.
```
    ResponseEntity
    .ok()
    .lastModified(1656759150L)
```

Burasi timestamp alıyordu unutmayın. Yani milisaniye cinsinden bir değer. (Burası da opsiyoneldir, yapanlar artı puan alacaktır.)

### Webservice 3 ###

İkinci webservisimiz hesaba para yukleyecektir arkadaşlar. Buraya yapılan istek JSON şeklinde sadece yüklenecek miktarı alacaktır.

```json
{
    "amount" : 100
}
```

Burada hangi account oldugu bilgisi sizinde bildiğiniz gibi URL'de gelecektir.(Path parametresi). Ve biz istek yapılan hesabın bakiyesini accounts.txt'de gelen amount parametresi kadar artıracağız arkadaşlar. Ve cevap olarak Account nesnesini full bir şekilde döneceğiz arkadaşlar.

```json
    {
        "accountNumber" : "2341231231"
        "name" : "Abdullah",
        "surname" : "Yavuz",
        "email" : "bootcamp@bootcamp.com",
        "tc" : "12121212121",
        "balance" : 100
        "type" : "TL"
    }
```

Buradada istek JSON gelecek. Buna göre uygun request sınıfını oluşturmanızı bekliyorum arkadaşlar. Cevap direk Account nesnesini döndüğü için onda özel bir sınıfa ihtiyacımız yoktur.

Bu işlem sonrası ayrı bir dosyaya(logs.txt)'ye log tutulmasını istiyorum arkadaşlar. Logun formatı
```
    hesap_no[space]operation_type[space]operation_detail
```
şeklinde olacaktır. Yani örnek bir log şu şekilde olacaktır;
```
    2341231231 deposit amount:100
```

Arkadaşlar bu bir update işlemidir temelde. Onun için uygun HTTP methodunu seçmenizi bekliyorum. (PATCH veya PUT)

### Webservice 4 ###

Bu servisimizde para transferi yapacağız arkadaşlar. Bir hesaptan diğer bir hesaba para aktarımı yapacağız.

Bu servisimizin isteği şu şekilde olacaktır olacaktır arkadaşlar.
```json
    {
        "transferredAccountNumber" : "4513423423",
        "amount" : 10
    }
```
Hangi accounttan transfer yapılacağı URL'den belirtilecek arkadaşlar.
Burada yapacağımız iş gönderilen hesaptan bakiyeyi düşmek, giden hesaptada bakiyeyi artırmaktır.

Bu işlem sonrasındada log tutmanızı istiyorum arkadaşlar. Ornek log şu şekilde olacaktır.
```
    2341231231 transfer amount:100,transferred_account:4513423423
```

#### ÖNEMLİ NOKTALAR ####
Burada dikkat edilecek iki nokta vardır.
* Gonderim yapan hesabın bakiyesinde yeterli miktar olmaması. Bu durumda kullanıcıya uygun HTTP status kodu ile
```json
    {
        "message" : "Insufficient balance"
    }
```
şeklinde mesaj dönmenizi istiyorum.

* İkinci nokta ise transfer yapılan hesapların türünün farklı olması durumunda uygun çevrimi yapmanız gerektiğidir. Yani mesela ben TL hesabından 100 TL'yi bir dolar hesabına transfer ediyorsam o hesabın balance'i dolar miktarından artırılmalıdır. Burada o anki kuru bir dış webservisten(***bir önceki ödevde kullandıgımız collectapi buna dair apilerede sahip, TL-USD veya TL-Altın dönüşümü yapan servisler var***) almanızı bekliyorum. Buna dair çok ornek var.(Bu zorunlu değil bir artıdır. Dolar kurunu sabit 17 TL, altının gramınıda sabit 1000 TL olarak alabilirsiniz.)

Eğer işlem başarılı olursa uygun HTTP kodu ile birlikte.
```json
    {
        "message" : "Transferred Successfully"
    }
```
şeklinde bir cevap dönmenizi bekliyorum arkadaşlar.


### Webservice 5 ###
4.cü webservisimiz bir accounta dair yapılan işlemleri dönecek web servisimizdir.
Bu servise PATH parametresi ile gelen accountNumber okunup logs.txt dosyasında bu accounta dair loglar alınacaktır.
Logları dönerken logu dosyada tuttugu formatta değil insan tarafından okunabilir bir şekilde dönecektir. Yani mesela
```
    2341231231 deposit amount:100
```

Bu logu dönerken 
  ```
    2341231231 nolu hesaba 100 [hesap tipi] yatırılmıştır.
  ```
Veya bunu dönerken  
```  
  2341231231 transfer amount:100,transferred_account:4513423423
```

```  
  2341231231 hesaptan 4513423423 hesaba 100 [hesap tipi] transfer edilmiştir.
```

şeklinde mesajlar verilecektir. Yani bu servisin cevabı şu şekilde
```json
[
    {
        "log" : "2341231231 hesaptan 4513423423 hesaba 100 [hesap tipi] transfer edilmiştir."
    },
    {
        "log" : "2341231231 nolu hesaba 100 [hesap tipi] yatırılmıştır."
    }
]
```

şeklinde JSON arrayi olacaktır.

# NOTLAR

* Arkadaşlar logları alabildiğimiz webservisin başka originlerdende ulaşılabilmesini istiyorum. Mesela siz servislerinizi htp://localhost:6161 de geliştirdiğiniz. Buraya http://localhost:6162 (browser ustunden) domaini altından yapılacak isteklerlede çalışabilmesini bekliyorum. (@CrossOrigin'i hatırlayın). Sadece bu servise özel bir durum, diğerlerinde böyle bir şey olmayacak.

* Arkadaşlar bankacılık sistemlerinde servis geliştirirken URL'leri REST'e uydurmak gerçekten zorlayıcı olabilir. Burada elinizden geleni yapmanızı istiyorum. Zorlandıgınız noktada bazı şartları esnetebilirsiniz.(URL'de action belirtilmez mesela)

* Arkadaşlar dosyada birşeyi güncellemek düşündüğünüz gibi olmayabilir. Dosyada gideyim üçüncü satırdaki bir kısmı güncelleyeyim diye bir olay yoktur. Yani bir hesabın bakiyesini güncellerken yepyeni bir dosyaya eski dosyada ne varsa, bir de güncellediğiniz veriyide yazmanız lazımdır. Bu işlem sonrasında elimizde güncellenmiş bir şekilde yenşi bir dosya olacaktır. İşin belkide en kritik kısmı burasıdır. Buna dikkat edin.

* Arkadaşlar loglama işlemi kafka tarafından yapılacaktır
* Kafka'da logs adında bir topic yaratmanızı ve logları buraya göndermenizi bekliyorum. Burada Kafka-spring entegrasyonu beklemiyorum düz java koduyla yapmanızı bekliyorum arkadaşlar. Dökumanda anlattıgım şekilde.
* Producer log messajını uygun bir şekilde kafkaya gönderecek ve bir adet consumer logları alıp dosyaya yazma işlemini gerçekleştirecektir arkadaşlar.
* Arkadaşlar adım adım çalışın, mesela once servisler yazılıp, sonra kafkaya bakılabilir. Daha sonra cors'a ve sondada account detayı getiren serviste LAST modified headerinin setlenmesine bakabilirsiniz.







