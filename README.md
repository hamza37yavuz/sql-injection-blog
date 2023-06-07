## **SQL INJECTION ZAFİYETİ**

İnternet tabanlı veri sistemlerinde yaygın bir zafiyet olan SQL enjeksiyonu (injection), uygulama geliştiricilerinin sıklıkla karşılaştığı bir güvenlik açığıdır. Günümüzdeki web uygulamalarının genelinin SQL enjeksiyonu zafiyeti sebebiyle güvenli olmadığı tahmin edilmektedir. Bu nedenle SQL enjeksiyonu zafiyetlerini öğrenmek ve onları engellemek, güvenli bir uygulama geliştirmenin önemli bir parçasıdır. Yazımızda bu zafiyeti 3 ana başlıkta ele alacağız:
### **1- SQL Enjeksiyonu Zafiyeti Nedir?**
![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/1.jpeg)

Bu zafiyet hakkında yazmadan önce SQL’in ne olduğunu anlamalıyız. SQL, Structured Query Language kelimelerinin kısaltması olan bir veritabanı yönetim dili olarak bilinmektedir. Bu dil, verilerin bir veri tabanında saklanmasını, değiştirilmesini, silinmesini ve sorgulanmasını sağlar.
SQL, yaygın olarak web uygulamalarının verilerini depolamak ve yönetmek için veri tabanı sistemleriyle birlikte kullanılır. Ancak, web uygulamalarının kullanıcı girdilerini doğrudan SQL sorgularına ekleyerek çalıştırması, SQL enjeksiyonu zafiyeti olarak adlandırılan ciddi bir güvenlik açığına yol açabilir.
SQL enjeksiyonu zafiyeti, saldırganın web uygulamasının veri tabanına zararlı SQL kodları enjekte ederek kötü amaçlar için kullanabileceği yetkiler elde etmesi ve veri tabanına sağladığı erişimle verileri manipüle etmesi anlamına gelir. Bu zafiyet, web uygulaması tarafından kullanıcı girdilerinin yeterince doğrulanmaması veya doğru şekilde işlenmemesi sonucu oluşur.
SQL enjeksiyonu zafiyeti genellikle üç ana kategoriye ayrılır:

![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/2.jpeg)

#### **In-band SQL Injection: (Ağ içinden)**
Bu tür enjeksiyonlarda saldırgan, veri tabanına doğrudan bağlantı kurarak saldırı gerçekleştirir. Örneğin web uygulamasında bir arama formu olduğunu varsayalım. Bu formda kullanıcıların arama yapabileceği bir alan mevcuttur. Eğer SQL enjeksiyonu zafiyeti varsa kötü niyetli kullanıcılar bu alana SQL kodu girerek verileri manipüle edebilirler.


#### **Out-of-band SQL Injection: (Ağ dışından)**
Bu tür enjeksiyonlarda saldırgan, hedef sisteme bağlı olmayan bir kanaldan saldırı gerçekleştirir. Örneğin, saldırgan, hedef sisteme dışarıdan bir e-posta göndererek saldırı yapabilir.


#### **Inferential SQL Injection: (Dolaylı SQL Enjeksiyonu)**
Veri tabanı yanıtı aracılığıyla çalışan bir SQL enjeksiyon tekniğidir. Bu teknikte, saldırganın gerçek zamanlı olarak yanıt alması gerekmez. Ancak yanıtları gözlemleyerek veri tabanındaki bilgileri keşfedebilir. Örneğin, bir e-ticaret web sitesinde ürün araması yapmak isteyen kötü niyetli bir kullanıcı olduğumuzu varsayalım. Linkte arama kelimesinin bir parametre olarak geçtiğini fark ettik, bu parametreyi kontrol ederek bir SQL enjeksiyonu yapabiliriz. (Kontroller iyi yapıldıysa böyle bir durumda her zaman SQL açığı olmayabilir.) Örneği biraz daha somutlaştıralım:
`https://www.example.com/search?q=book` şeklinde bir link olduğunu görürsek bu linki aşağıdaki şekilde değiştirerek veriler hakkında bilgi sahibi olabiliriz: `https://www.example.com/search?q='; IF (SELECT price FROM products WHERE id=1) < 100 THEN pg_sleep(10) ELSE pg_sleep(0); -- `


Kısacası SQL enjeksiyonu zafiyeti, kötü niyetli saldırganın web uygulamasına gönderdiği SQL sorgusu ile veri tabanına izinsiz erişim sağlamasına ve verileri manipüle etmesine olanak tanıyan bir güvenlik açığıdır.

### **2- SQL Enjeksiyonu Nasıl Yapılır? (Uygulamalı)**
SQL enjeksiyonunu uygulamalı olarak gösterebilmek için online bir laboratuvardan faydalanmayı uygun bulduk. Bu [linke](https://portswigger.net/web-security/sql-injection/lab-login-bypass) tıklayıp laboratuvara erişebilirsiniz. Laboratuvara girmeden önce hesabınız varsa giriş yapmanız hesabınız yoksa hesap oluşturmanız gerekiyor. Bu kısımları yaptıktan sonra zafiyetli web sitesini görüntüleyeceksiniz. Bu kısımdan sonrasını aşağıda adım adım fotoğrafları ekleyerek anlatacağım.
Adımlara geçmeden önce linke tıkladıysanız karşınıza çıkan sayfadaki bilgilendirmeyi okumayı unutmayın.

#### **a) Login Kısmını Bulalım ve Biraz Kurcalayalım:**
Zafiyetli e-ticaret sitesine girdikten sonra sağ üstteki benim hesabım (my account) kısmına tıklayarak giriş sayfasına erişim sağlıyoruz. Giriş sayfasına eriştiğimizde ilk aklımıza gelen varsayılan kullanıcı adı ve şifreyi deniyoruz. (Bu adımı ben hiçbir zaman atlamam ama isterseniz siz atlayabilirsiniz.) Laboratuvara girmeden önceki bilgilendirmede verilen ‘administrator’ bilgisini hem kullanıcı adı hem de şifre kısmına yazarak bir deneme yapıyoruz. Bu kadar kolay olmadığını tahmin etmeliydik. Daha fazla deneme yapmadan önce SQL enjeksiyonu zafiyetine yönelik denemeler yapalım. Öncelikle varsayılan kullanıcı adı ve parola için bir SQL sorgusunun nasıl olacağını düşünelim:

`SELECT * FROM users WHERE username ='administrator’ AND password = 'administrator'`

Sorgunun yukarıdaki gibi olabileceğini düşünerek yolumuza devam edelim. Bu sorgu users (veritabanındaki bir tablo ismidir, tablo isimleri örnek olarak verilmiştir) tablosu içerisinden username ve password değerleri administrator olan bir veri arıyor. Tabloda verileri bulup bulamamasına göre dönüş yapıyor. Eğer SQL için bir önlem alınmadıysa ‘ işareti gibi bir kullanıcı adı girişimiz karşısında sitenin sunucu hatası vermesini bekleriz. Çünkü SQL sorgumuz aşağıdaki şekilde olur:

`SELECT * FROM users WHERE username =’’’ AND password = 'administrator' ` 

Böyle bir sorgu SQL yazım dili için bir hata teşkil etmektedir. Laboratuvarda kullanıcı adı kısmına ‘ işareti girerek deneyelim. Bakalım sonuç ne olacak: 

![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/3.jpeg)
![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/4.jpeg)

SQL açığı olduğunu kesin olarak anladığımıza göre ikinci ve son adıma geçelim.

#### **b) Administrator Olarak Giriş Yapma:** 
İlk adımda bir SQL enjeksiyonu zafiyeti olduğunu öğrendik. Administrator olarak girişi yapabilmek için nasıl SQL sorgusu olmalı onu düşünelim. Hem kullanıcı adını hem de şifreyi bilmediğimiz bir senaryo olduğunu varsayalım. Bu durumda kullanıcı adını ve şifre kısmını atlatacak bir SQL sorgusuna ihtiyacımız var. Kullanıcı adı ve şifreyi boş bırakarak giriş yapmaya çalıştığımızda veri tabanına aşağıdaki sorgu yapılır:

`SELECT * FROM users WHERE username =' ’ AND password = ' '`

SQL dilinde yazdığımız sorgulardan herhangi birinde yorum satırı kullanmak için – ya da # işaretini kullanırız. Şifre kısmındaki SQL sorgusunda atlayabilmek için yorum satırı ve her zaman girilen verileri doğru olarak gören SQL sorgusu elde etmek için bir mantık ifadesi olan OR kullanmamız gerekecek. Nasıl bir SQL sorgusu yapmamız gerektiğine bakalım:

`SELECT * FROM users WHERE username ='’ OR 1=1 – ‘ AND password = ''`

Yukarıdaki sorguda sadece kalınlaştırılmış olan kısmın sorgusu yapılacaktır. Çünkü devamındaki kısmın yorum satırı olması sağlanmıştır. OR ifadesi ile SQL sorgusunda başarılı olması için 1=1 yazdık. Aşağıdaki fotoğraflara bakarak bu SQL sorgusunu giriş ekranında nasıl yapabileceğimize bakalım:

![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/5.jpeg)
![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/6.jpeg)

Tebrikler administrator olarak giriş yapmayı başardık. Kullanıcı adına administrator yazmadan administrator olarak giriş yapabildiğimizi fark etmişsinizdir. Bunun sebebi veri tabanının böyle bir SQL açığında kullanıcı adı yazmadan doğru SQL sorgusu yaparsanız sizi varsayılan olarak administrator kullanıcısına aktaracak şekilde ayarlanmış olmasıdır.

Bu laboratuvar farklı şekillerde çözülebilirdi. Size önerim bu laboratuvarı farklı şekilde çözerek ya da bu [linkteki](https://portswigger.net/web-security/sql-injection/) diğer SQL laboratuvarlarını çözerek bu konudaki bilgilerinizi artırmanızdır.

### **3- SQL Enjeksiyonu Zafiyetine Nasıl Önlem Alınabilir?**
Bir etik üzerine bu zafiyetleri araştıracağımızı ve bu zafiyeti düzeltme konusunda neler yapılacağını da bilmemiz gerektiğini unutmayalım. SQL enjeksiyonu zafiyetinin ne olduğunu nasıl yapılabileceğini konuştuk. Şimdi web geliştirici ya da veri tabanı yöneticisi olduğumuzu düşünelim ve buna nasıl önlemler alabileceğimizi 4 ana başlıkta ele alalım:

#### **a) Parametrelerin Doğrulanması:**
Geliştiriciler (yani biz), kullanıcıların girdilerini doğrulayarak SQL enjeksiyonu saldırılarını önleyebiliriz. Bu sebeple veri tabanı sorgularına gönderilen parametrelerin türünü ve değerini doğrulamalıyız ve sorgularda kullanılmaları gereken parametrelerin sayısını sınırlandırmalıyız.

![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/7.jpeg)

Örneğin, bir web formuna kullanıcının telefon numarasını girdiğini varsayalım. Bu bilginin tam sayı tipinde olduğunu doğrulayabiliriz. Kullanıcının telefon numarası bilgisinin tam sayı olmadığı tespit edildiğinde, bu girdileri veri tabanına göndermeyiz ve kullanıcıya yukarıdaki gibi uyarı gösteririz.

#### **b) Parametreleri Veritabanı Sorgularında Kullanmama:**
SQL enjeksiyonu saldırılarının önlemek için kullanabileceğimiz en iyi yöntem, parametreleri veritabanı sorgularında doğrudan kullanmamaktır. Bu nedenle, veritabanı sorgularında parametrelerin yerini belirleyen belirleyicileri kullanmalıyız.
Örneğin, veri tabanına bir kullanıcının adını ve şifresini sorgulama isteği gönderildiğini varsayalım. Şifrenin doğrudan sorguda kullanılması yerine, sorguda belirleyiciler kullanmalıyız. Bu, saldırganların doğrudan şifre değerlerini sorgulamalarına veya değiştirmelerine olanak tanımaz.

#### **c) Veritabanı Kullanıcı İzinleri:**
Veritabanı yöneticisi olarak bizler, veritabanı kullanıcılarının izinlerini kısıtlayarak SQL enjeksiyonu saldırılarını önleyebiliriz. Kullanıcıların sadece belirli veritabanı sorgularını çalıştırmalarına izin vermemiz, saldırganların sorgulamaları manipüle etmelerini önler.
Örneğin, bir veritabanı yöneticisi aşağıdaki SQL kodunu kullanarak "personel" adlı bir kullanıcı oluşturabilir ve sadece "calisanlar" adlı bir tablodaki verileri görüntülemesine izin verebilir:

`CREATE USER 'personel'@'localhost' IDENTIFIED BY 'Sifre123';`
`GRANT SELECT ON mydb.calisanlar TO 'personel'@'localhost';`

Bu şekilde, "personel" kullanıcısı sadece "calisanlar" tablosundaki verileri görüntüleyebilir ve diğer tablolara veya veri tabanı nesnelerine erişemez. Bu, kullanıcıların sadece ihtiyaç duydukları veritabanı nesnelerine erişebileceği ve zararlı işlemleri yapamayacağı anlamına gelir.

#### **d) Güvenlik Duvarı (Firewall) Kullanımı:**
![](https://github.com/hamza37yavuz/sql-injection-blog/blob/main/8.jpeg)

Güvenlik duvarı, bir ağda gelen ve giden trafikleri izler ve zararlı trafikleri engeller. SQL enjeksiyonu gibi saldırılar için de bir güvenlik duvarı kullanabiliriz. Bu tip güvenlik duvarları, SQL sorgularını filtreleyerek zararlı girişlerin önüne geçebilir.
Bunun için, uygulama sunucusu ve veritabanı sunucusu arasında bir güvenlik duvarı oluşturabiliriz. Bu güvenlik duvarı, gelen SQL sorgularını analiz eder, beklenmedik karakter içeren veya tehlikeli olan sorguları engelleyebilir. Ayrıca güvenlik duvarını, sadece belirlenen parametreler ve değerlerin SQL sorgusuna ekleyecek şekilde ayarlayarak güvenlik seviyemizi artırabiliriz.


Sonuç olarak, SQL enjeksiyonu zafiyetlerinin önlemek için kullanabileceğimiz en iyi yöntem, uygulama kodunda gerekli güvenlik önlemlerini almamızdır. Bunun yanı sıra, güvenlik duvarı gibi ek önlemler de alarak, uygulamanın güvenliğini artırabiliriz. Hangi tarafta (siber güvenlik ya da web geliştirme) olursak olalım web uygulamalarının daha iyi çalışabilmesi ve verilerin güvenliği için çalıştığımızı unutmayalım.


Bu yazımızda SQL enjeksiyonu zafiyetinin ne olduğundan bu zafiyetin nasıl sömürülebileceğinden ve bir geliştiricinin bu zafiyetlere ne gibi önlemler alabileceğinden bahsettik.
Başka bir yazıda görüşmek dileğiyle 😊

#### **KAYNAKÇA:** 
>https://yakupseker.medium.com/sql-injection-nedir-nas%C4%B1l-bulunur-mutillidea-sql-injection-cevaplar%C4%B1-level-low-c6eff4e1e09

>https://www.acunetix.com/websitesecurity/sql-injection2/

>https://serverfault.com/questions/329098/is-there-a-way-to-make-sql-browser-service-work-behind-isa-firewall-what-would

>https://portswigger.net/web-security/sql-injection

>https://portswigger.net/web-security/sql-injection/lab-login-bypass 

>https://sectigostore.com/blog/what-is-sql-injection-8-tips-on-how-to-prevent-sql-injection-attacks/

