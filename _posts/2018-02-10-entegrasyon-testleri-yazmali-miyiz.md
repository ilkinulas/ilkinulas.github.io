---
layout: post
title: Entegrasyon Testleri Yazmalı Mıyız?
categories: development test junit docker
---
Yeni bir yazı serisine başlıyorum. Bu seride entegrasyon testlerini `JUnit` ve `Docker` yardımıyla nasıl yazdığımı anlatacağım. Serinin ilk bölümü olan bu yazıda entegrasyon testlerinin ne olduğundan ve neden entegrasyon testi yazmamız gerektiğinden bahsedeceğim. 

* JUnit `@Rule`'larından bahsettiğim serinin ikinci yazısını [buradan okuyabilirsiniz](/development/test/junit/docker/2018/02/20/Junit-rules.html).
* Docker ve Docker Compose'dan bahsettiğim serinin üçüncü yazısını [buradan okuyabilirsiniz](/development/test/junit/docker/2018/03/11/docker-ve-docker-compose.html).
* Serinin dördüncü yazısı TestContainers'ı da [buradan okuyabilirsiniz](/development/test/junit/docker/2018/03/18/testcontainers.html).

9 Eylül 1947'de (71 yıl önce) Tümamiral ve aynı zamanda bir bilim kadını olan [Grace Hopper](https://www.biography.com/people/grace-hopper-21406809) bilgisayar tarihinin ilk bug'ını 🐞 kayıt altına [almış](http://www.computerhistory.org/tdih/September/9/). Aşağıdaki resimde Grace Hopper'ın el yazısı ile *Harvard Mark II* adındaki (elektro-mekanik) bilgisayarın kayıt defterine yazılmış hata raporunu görüyorsunuz.

![First ever bug](/assets/integration_tests/first_recorded_bug.jpg){: .center-image }

Program hataları 🐞 yıllardır hayatımızın bir parçası. Aslında _program hatası_ değil _programcı hatası_ demek daha doğru olacak. Sanki biz hatasız ve düzgün çalışan bir program yazıyormuşuz da "*program*" sonradan kendi kendisine bozulup hatalı çalışmaya başlıyormuş anlamına geliyor "_program hatası_" demek. Neyse; __hatasız program olmaz, fakat daha az hatalı programlar üretmek bizim elimizde.__

Yazılımlarımızı müşterileri ile buluşturmadan önce test ederiz. Kimse müşterisine çalışmayan ya da yanlış çalışan bir yazılım teslim etmek istemez. Rekabetin hiçbir zaman azalmadığı, giderek arttığı bilişim dünyasında, yazılımın kalitesinden ödün vermeden hızlı bir şekilde ürünleri piyasaya sürmek çok önemlidir. Onlarca tester 👨‍💼 👩‍💼 👨‍💼 çalıştırıp uygulamaların testlerini manuel yapmak günümüzün hızlı yazılım üretme ihtiyacını karşılamaz, çünkü manuel testler hem çok masraflı 💸 hem de geri dönüşü çok geç olan testlerdir.

Yeni bir yazılım projesine başladığımızı hayal edelim. Geliştirme aşamasının başlarında herşey düzgün çalışıyor ve olan az sayıdaki özelliği de elimizle test edebiliyoruz. Projenin ilk fazı tamamlandı ve ürünü müşteriye teslim ettik. Bir süre sonra müşterimiz bize ürün ile ilgili bir hata raporladı. Hatayı düzelttik, mevcut test senaryolarımız arasına son hataya sebep olan durumu da ekledik ve tüm test senaryolarının tekrar üzerinden geçtik ve yeni bir release çıkarttık. Zamanla ürüne yeni özellikler eklememiz gerekti, __çünkü değişim yazılımın doğasında var.__ Yeni özellikleri ekledik ve eklenen bu özelliklerle birlikte test edilmesi gereken senaryoların sayısı da arttı. Artık ürünü test etmek için saatler 🕑 harcamamız gerekiyor. Ürünümüz çok başarılı oluyor ve başka müşteriler de bizim ürünümüzü kullanmak istiyorlar Fakat bu yeni müşteriler ürünümüzü farklı bir işletim sisteminde çalıştırmak istiyorlar. Ürünü farklı işletim sistemlerinde de çalışacak hale getiriyoruz. Ürünün kalitesinden emin olmak için aynı testleri farklı işletim sistemleri için tekrar tekrar yapmamız gerekiyor. Neyse lafı çok fazla uzatmayayım, her yeni release'den önce tüm test senaryolarını tek tek çalıştırıp uygulamanın hala beklediğimiz gibi hatasız çalıştığından emin olmak bir süre sonra sürdürülemez bir hal alıyor. 😕

Uygulamamızda yaptığımız kod ya da konfigürasyon değişikliklerinin yeni bug'lara 🐛 sebep olup olmadığını anlamak için yapılan testlere havalı bir isim verilmiş. [`Regression Tests`](https://stackoverflow.com/questions/3464629/what-does-regression-test-mean). İlk başlarda regresyon testlerini manuel yapabiliyorduk çünkü uygulamamız küçüktü ve çok fazla test edilmesi gereken senaryo yoktu. Zamanla testlerin sayısı arttı ve farkettik ki vaktimizin çoğunu yazdığımız kodların doğruluğunu test etmek için harcıyoruz.

Yazılımlarımızın hatasız çalıştığından emin olmak için manuel regresyon testleri yapabiliriz fakat değerli zamanımızı bu testleri tekrar tekrar çalıştırarak harcamanın iyi bir fikir olduğunu düşünmüyorum. Peki ne yapmalıyız? 

> Testlerimizi hızlı ve otomatik çalıştırılabilir hale getirmeliyiz.

## Automated Tests

Otomatik çalıştırılabilir testleri (`Automated Tests`) en güzel aşağıki `Test Piramidi` çizimi özetler. 

![Test Pyramid](/assets/integration_tests/test_pyramid.jpg){: .center-image }

Test piramidinde yukarılarara çıkıldıkça testlerin maliyeti 🕑 💸 artar. Aşağılara inildikçe de testlerin hızı 🏎️ artar. Bu piramidin bize verdiği mesaj şu : __mümkün olduğunca çok unit testiniz olsun, gereği kadar da entegrasyon testi yazın.__

## Unit Tests

Unit testler adı üzerinde uygulamamızdaki bir birimi test eder. Functional bir programlama dili kullanıyorsak bu birim bir fonksiyon olabilir. Object Oriented bir programlama dili kullanıyorsak bu birim bir method hatta büyük bir class olabilir. 

> Unit testleri çok seviyorum çünkü unit testler bizi uygulamamızın tasarımını iyileştirmemiz konusunda sürekli zorlar.

Her şeyin iç içe geçtiği, en ufak bir soyutlamanın olmadığı, objelerin çalışmak için başka onlarca objeye ihtiyaç duyduğu projeler görmüşsünüzdür. Böyle bir projede unit test yazmak çok zordur. Unit test yazabilmek için `refactoring` yapmamız gerekir. Class'larımızı [`Single Responsibility`](http://www.kurumsaljava.com/2009/10/14/single-responsibility-principle-srp-tek-sorumluk-prensibi/) prensibine göre tasarlayıp, sınıflarımızın bağımlılıklarını soyutlarsak (abstraction) class'lar daha kolay test edilebilir hale gelir. Yani __test edilebilirlik ve iyi kod tasarımı arasında bir sinerji vardır.__

## Integration Tests

Fakat unit testler tek başına kaliteli yazılım üretmek için yeterli olmazlar. Bu konuyla ilgili çok hoşuma giden aşağıdaki gif'leri paylaşmazsam olmaz. 😂😂😂

|     |      |
|-----|------|
![Two unit tests](/assets/integration_tests/no_integration_test.gif){: .center-image }|![Two unit tests](/assets/integration_tests/two_unit_tests.gif){: .center-image }

Tipik bir uygulama database, filesystem ve network bağlantısına ihtiyaç duyar. Unit testler hızlı ve izole çalışmaları gerektiği için bu bağımlılıkları test etmezler. Bu yüzden unit testler tek başına uygulamanın doğru çalıştığını gösterme konusunda yeterli olmazlar. Class'ın, modülün ya da tüm uygulamanın entegre olduğu servisler ile beraber düzgün bir şekilde çalıştığını test etmek için `entegrasyon testleri` yazarız. Entegrasyon testleri, test piramidinde unit testlerin üzerinde yer alır. Çünkü:

* Entegrasyon testleri unit testlere göre daha __yavaş çalışırlar__. Çalışmak için bir database'e ya da network üzerinden erişebileceği bir mikroservise ihtiyac duyabilirler.
* Entegrasyon testleri __kırılgan olurlar__. Çok fazla bağımlılığı bir arada test ettikleri için hata ile karşılaşabilme  ihtimali yüksektir. Unit testler izole çalıştıkları için başka servislerde meydana gelebilecek hatalardan etkilenmezler.
* Entegrasyon testlerini yazması ve __`debug` etmesi zordur__. Entegrasyon testi birden fazla bağımlılığı test ettiği için `fail` ettiğinde hatanın nerede olduğunu anlamak zordur.

Bu sebeplerden dolayı çok sayıda unit test ve yeteri kadar entegrasyon testi kaliteli yazılım için iyi bir formüldür.

JUnit ve Docker kullanarak entegrasyon testlerinin nasıl yazılabileceğini anlatacağım serinin giriş yazısının sonuna geldik. `Entegrasyon testi yazmalı mıyım?` sorusuna cevabın `Evet` ise takipte kal.