---
layout: post
title: TestContainers
categories: development test junit docker
---
Entegrasyon testleri yazı serisinin 4. bölümünde open source bir proje olan [TestContainers](https://www.testcontainers.org/)'ı tanıtmak istiyorum.

Serinin önceki yazılarını okumak isterseniz:
1. [Entegrasyon Testleri yazmalı mıyız?](/development/test/junit/docker/2018/02/10/entegrasyon-testleri-yazmali-miyiz.html)
2. [JUnit @Rule'ları](/development/test/junit/docker/2018/02/20/Junit-rules.html)
3. [Docker ve Docker-Compose](/development/test/junit/docker/2018/03/11/docker-ve-docker-compose.html)

![Docker Logo](/assets/testcontainers/logo.png)

TestContainers, JUnit ve Docker kullanarak entegrasyon testleri yazmanızı sağlayan bir java kütüphanesidir. JUnit class ve method rule'ları ile, testlerden önce container'ları başlatıp testler bitince container'ları temizler. 

TestContainers, docker daemon ile haberleşmek için [docker-java](https://github.com/docker-java/docker-java) kütüphanesini kullanır. `docker-java`, komut satırından çalıştırdığınız docker client ile yapabileceğiniz herşeyi Java API'ları ile yapabilmenizi sağlar.

Docker container'ları başlatmanın birden fazla yöntemi var. Örneğin mysql database'ini başlatmak için aşağıdaki gibi `docker run` komutunu çalıştırabiliriz.

```
$ docker run --name database -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_DATABASE=tododb \
    -e MYSQL_USER=todouser \
    -e MYSQL_PASSWORD=todopass \
    -d mysql:5.7.21
```

Ya da `docker-compose` komutu ile [docker-compose.yml](https://github.com/ilkinulas/kotlin-todo-app/blob/master/docker-compose.yml) dosyasında tanımlı `database`'i  aşağıdaki gibi başlatabiliriz.

```
$ docker-compose up database
```

Yukarıdaki iki örneğin TestContainers'taki (kotlin) karşılığı aşağıdaki gibidir:

{% highlight kotlin %}
@ClassRule @JvmField
val database = KGenericContainer("mysql:5.7.21")
        .withExposedPorts(3306)
        .withEnv("MYSQL_ROOT_PASSWORD", "root")
        .withEnv("MYSQL_DATABASE", "tododb")
        .withEnv("MYSQL_USER", "todouser")
        .withEnv("MYSQL_PASSWORD", "todopass")
{% endhighlight %}

TestContainers'ın özelleşmiş container sınıflarını kullanarak mysql sunucusunu testlere hazır hale getirmek daha kolay. Ortam değişkenlerini `withEnv` metodu ile belirtmek yerine container DSL'i kullanabiliriz.

{% highlight kotlin %}
@ClassRule @JvmField
val database = KMysqlContainer("mysql:5.7.21")
        .withDatabaseName("tododb")
        .withUsername("todouser")
        .withPassword("todopass")
{% endhighlight %}

[GenericContainer](https://github.com/testcontainers/testcontainers-java/blob/master/core/src/main/java/org/testcontainers/containers/GenericContainer.java) yerine özelleşmiş [MySQLContainer](https://github.com/testcontainers/testcontainers-java/blob/master/modules/mysql/src/main/java/org/testcontainers/containers/MySQLContainer.java) kullanmanin bazı avantajları var.

* MySQLContainer, mysql database'inin bağlantı kabul edip sorgulara cevap vermesini bekler. Bu sayede testler çalışmaya başladığında mysql database'inin kullanıma hazır olur.
* MySQLContainer'ın API'si daha anlaşılır. `withUsername`, `withPassword`, `withDatabase` gibi methodları sayesinde ortam değişkenleri (ENV) ile mysql database'ini ayarlamamıza gerek kalmıyor.
* Mysql'e özel olduğu için mysql'in defaul çalışma portu `3306` otomatik olarak `expose` ediliyor.

[H2](http://www.h2database.com/html/main.html), [hsqldb](http://hsqldb.org/) gibi in-memory ve çok hızlı çalışan database'ler varken neden gerçek bir database sunucusu kullanıp testlerimi yavaşlatayım diye düşünebilirsiniz. Entegrasyon testlerinde gerçek database'ler (MySQL, PostgreSQL, Oracle XE) kullanmanın hem avantajları hem de dezavantajları var.

#### Avantajları
* Herşeyden önce testler production ortamı ile aynı ayarlara sahip database sunucuları ile çalışmış oluyor.
* H2 ve hsqldb gibi in-memory database'ler production ortamında kullandığımız gerçek database'lerin tüm özelliklerini içermeyebilirler. Örneğin Oracle DB'de kullandığımız `stored procedure`'leri testler için kullandığımız in-memory database engine için ayrıca kodlamamız gerekebilir. Ya da MySQL'deki [`UNIX_TIMESTAMP()`](https://dev.mysql.com/doc/refman/5.5/en/date-and-time-functions.html#function_unix-timestamp) fonksiyonunun H2'de olmaması gibi uygulamanın ihtiyacı olan başka birçok özellik in-memory database'de eksik olabilir.

#### Dezvantajları
* Production ortamında kullandığımız MySQL, PostgreSQL gibi database'lerin başlatılması ve testler bitince kapatılması in-memory database'ler kadar hızlı olmaz.

TestContainers kütüphanesinin kullanımını göstermek için Kotlin ile basit bir web uygulaması geliştirdim. Uygulamanın kaynak kodlarına [kotlin-todo-app](https://github.com/ilkinulas/kotlin-todo-app) github reposundan erişebilirsiniz. Üç katmanlı bu basit web uygulamasında:

* Ön yüzde [vuejs](https://vuejs.org/) kullanıldı.
* Backend [SparkJava](http://sparkjava.com/) web framework'u kullanılarak geliştirildi.
* Data storage katmanında ise MySQL ( ya da başka bir relational database) kullanıldı.

Bu uygulamanın end-to-end testlerini TestContainers yardımıyla yapmaya çalışalım. Test setup'ımız aşağıdaki resimdeki gibi olacak. 
* MySQL database container başlatacağız.
* Daha önceden docker imajını hazırladığımız todo uygulamamızı başlatacağız. Demo uygulamasının docker imajına [https://hub.docker.com/r/ilkinulas/todoapp/](https://hub.docker.com/r/ilkinulas/todoapp/) adresinden erişebilirsiniz.
* End-to-end testlerin otomasyonu için de [selenium](https://www.seleniumhq.org/) çalıştıran bir container başlatacağız. 

![End To End Test Setup](/assets/testcontainers/endtoendtestsetup.jpg){: .center-image }

TestContainers kütüphanesini kotlin ile kullanmak isterseniz aşağıdaki gibi wrapper container class'larınızı yaratmanız gerekiyor.
Bu konuyla ilgili [github issue'sunu](https://github.com/testcontainers/testcontainers-java/issues/318) inceleyebilirsiniz.

{% highlight kotlin %}
class KMysqlContainer(dockerImage: String) : MySQLContainer<KMysqlContainer>(dockerImage)
class KGenericContainer(dockerImage: String) : GenericContainer<KGenericContainer>(dockerImage)
class BrowserContainer : BrowserWebDriverContainer<BrowserContainer>()
{% endhighlight %}

Şimdi sırasıyla container'larımızı teste hazırlayalım. 

{% highlight kotlin %}
val database = KMysqlContainer("mysql:5.7.21")
        .withNetwork(network)
        .withNetworkAliases("database")
        .withDatabaseName("tododb")
        .withUsername("todouser")
        .withPassword("todopass")
{% endhighlight %}

MySQL container'ı oluştururken `"mysql:5.7.21"` parametresi kullandım. Çünkü testlerimin her zaman aynı mysql sunucusu versiyonu ile çalışmasını istiyorum. Sadece `"mysql"` yazsaydım, TestContainers `"mysql:latest"` imajını kullanacaktı. Container imajlarının yeni versiyonları çıkınca sürprizlerle karşılaşmamak için sabit bir versiyon kullanmayı tercih ediyorum.  MysqlContainer'i oluştururken kullandığım `withNetwork` ve `withNetworkAliases` metodlarının ne işe yaradığını birazdan anlatacağım.

{% highlight kotlin %}
val todoapp = KGenericContainer("ilkinulas/todoapp:1.0")
        .withNetwork(network)
        .withNetworkAliases("todoapp")
        .withExposedPorts(9000)
        .withEnv("DB_URL", "jdbc:mysql://database:3306/tododb")
{% endhighlight %}

Todo Web uygulamasının container'ını da [GenericContainer](https://github.com/testcontainers/testcontainers-java/blob/master/core/src/main/java/org/testcontainers/containers/GenericContainer.java) sınıfı yardımıyla başlatıyorum. Demo uygulamam 9000 portundan hizmet verdiği için container'ın bu portu dinlemesini söylüyorum. `withEnv` metodu ile de container çalışırken `DB_URL` ortam değişkenine istediğim bir değer atıyorum.

{% highlight kotlin %}
val browser = BrowserContainer()
        .withNetwork(network)
        .withNetworkAliases("browser")
        .withDesiredCapabilities(DesiredCapabilities.chrome())
        .withRecordingMode(BrowserWebDriverContainer.VncRecordingMode.RECORD_ALL, File("./out/"))
{% endhighlight %}

Selenium testlerimi çalıştırmak için de bir tane [BrowserWebDriverContainer](https://www.testcontainers.org/usage/webdriver_containers.html)'a ihtiyacım var. Bu container sınıfı da MysqlContainer gibi özelleşmiş bir sınıf. End-to-end testlerimin hepsinin `./out/` dizinine kaydedilmesi için `withRecordingMode`'nu kullandım. Bu sayede testlerden birisi başarısız olduğunda kaydedilen test videosu hatanın kaynağını bulmaya yardımcı olacak.

Bu üç container'ın birbirlerine erişebilmesi için docker'ın [networking altyapısını](https://docs.docker.com/network/network-tutorial-standalone/) kullandım. TestContainers'ın [@Network](https://github.com/testcontainers/testcontainers-java/blob/master/core/src/main/java/org/testcontainers/containers/Network.java) rule'u ile aşağıdaki gibi bir network oluşturuyorum.

{% highlight kotlin %}
val network = Network.newNetwork()
{% endhighlight %}

Tüm container'larin aynı network'ü kullanabilmesi için `withNetwork(network)` metodunu kullandım. `withNetworkAliases(alias:String)` metodu ile de container'ın network içinde hangi hostname ile erişilmesini istediğimi belirtiyorum. Örneğin mysql container için  network alias olarak "database" kullandım. Bu sayede web uygulaması container'ımdan mysql'e `jdbc:mysql://database:3306/tododb` URL'ı ile erişebiliyorum. Küçük test network'üm için DNS ayarlarını yapıyorum gibi düşünebilirsiniz.

Son olarak RuleChain ile bu Rule'ların hangi sırayla çalışması gerektiğini belirtiyorum. Önce test network'üm oluşturulacak. Sonra mysql sunucusu başlatılacak. Mysql 3306 portundan bağlantı kabul etmeye başladıktan sonra demo web uygulamam başlayacak. RuleChain'de son sırada e2e testleri çalıştırmak için kullanılan selenium container başlatılacak.
{% highlight kotlin %}
@ClassRule
@JvmField
val ruleChain = RuleChain.outerRule(network).around(database).around(todoapp).around(browser)
{% endhighlight %}

`EndToEndTest` sınıfının tamamına [bu github linki](https://github.com/ilkinulas/kotlin-todo-app/blob/master/src/test/kotlin/net/ilkinulas/EndToEndTest.kt) üzerinden ulaşabilirsiniz.
