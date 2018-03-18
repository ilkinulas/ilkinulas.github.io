---
layout: post
title: Docker ve Docker-Compose
categories: development test junit docker
---
JUnit ve Docker yardımıyla entegrasyon testleri yazmayı anlattığım yazı serisinin 3. bölümünde `docker` ve `docker-compose`'dan bahsedeceğim. Test otomasyonunun öneminden bahsettiğim serinin birinci yazısına [buradan](/development/test/junit/docker/2018/02/10/entegrasyon-testleri-yazmali-miyiz.html) ve JUnit @Rule'larını anlatan serinin ikinci yazısına da [buradan](/development/test/junit/docker/2018/02/20/Junit-rules.html) erişebilirsiniz.

Docker, uygulama geliştirmeyi, deploy etmeyi ve çalıştırmayı kolaylaştıran açık kaynak kodlu bir platformdur. Go programlama dili ile geliştirilmiş bu teknolojinin resmi web sitesindeki [kaynaklar](https://docs.docker.com/) ingilizce bilenler için gayet yeterli. Türkçe kaynak arayanlar için de [Gökhan Şengün](https://www.gokhansengun.com)'ün Docker [yazı dizisini](https://www.gokhansengun.com/docker-nedir-nasil-calisir-nerede-kullanilir/) okumalarını öneririm. Bu kaynakların üzerine ekleyebileceğim birşey olduğunu düşünmediğim için Docker'ın yazılım geliştiriciler için sağladığı kolaylıkları bir örnek üzerinden anlatmaya çalışacağım.

![Docker Logo](/assets/docker/docker_logo.png)

Bu yazı için basit bir `todo web uygulaması` hazırladım. Uygulamanın kaynak kodlarına [github.com/ilkinulas/kotlin-todo-app](https://github.com/ilkinulas/kotlin-todo-app) adresinden erişebilirsiniz.

Docker ile birlikte uygulamaları deploy etme şeklimiz değişti. Docker'dan önce, uygulamalarımızı paketleyip çalışacakları ortama kopyalıyorduk ve bu ortamda uygulamanın çalışması için gerekli tüm ayarların yapılmış olmasını ve uygulamanın ihtiyacı olan tüm kütüphanelerin ortamda erişilebilir olmasını bekliyorduk. Java dünyasından bir örnek verecek olursak, uygulamamızı deploy ettiğimiz ortamda/host'ta bizim uygulamamız ile uyumlu bir java virtual machine'nin kurulu olmasını bekleriz. 

## Container Image
Geliştirdiğimiz uygulamayı, uygulamayı çalıştıracak runtime'ı, sistem araçlarını, uygulamanın ihtiyacı olan sistem kütüphanelerinin hepsini içeren çalıştırılabilir pakete Docker dünyasında `Container Image` diyoruz. Container image'larının başlıca özellikleri şunlardır:
* Çalıştıkları makinenin işletim sistemi kernel'ini ve kaynaklarını kullandıkları için çok hızlı başlatılabilirler. Virtual machine'lere göre daha az kaynak tüketirler.
* Standartlara dayandıkları için hemen hemen tüm işletim sistemlerinde çalıştırılabilirler.
* Container'lar içinde çalışan uygulamalar host işletim sisteminden ve host üzerinde çalışan diğer uygulamalardan izole çalışırlar. Bu izolasyon uygulama güvenliği sağlar.

## Dockerfile
Container image'ları build etmek için `Dockerfile` yazarız. Dockerfile'lar içerisinde image'in nasıl oluşturulması gerektiği ile ilgili komutlar bulunur. Aşağıda demo uygulamasının docker imajını oluşturmak için kullandığım Dockerfile'ı görüyorsunuz:

<script src="http://gist-it.appspot.com/http://github.com/ilkinulas/kotlin-todo-app/blob/master/Dockerfile"></script>

Teker teker Dockerfile içerisindeki komutları inceleyelim:
### 1. FROM
`FROM` komutu ile oluşturacağımız imajın baz alacağı bir imaj seçeriz. Java uygulamaza bir imaj hazırladığımız için `openjdk:8-jre-alpine` imajını baz aldık. Uygulamamızı [Alpine Linux](https://alpinelinux.org/) üzerine kurulu OpenJDK ile çalıştıracağız.

### 2. COPY
COPY komutu container içerisine dosya kopyalamak için kullanılır. Yukarıdaki örnekte `./gradlew clean shadowJar` komutu ile oluşturulan `jar` dosyası container dosya sisteminde `/opt/todo`altına kopyalanıyor.

### 3. WORKDIR
WORKDIR komutu kendisinden sonra çalıştırılacak komutlar için `working directory` ayarlamak için kullanılıyor. Bizim örneğimizde `ENTRYPOINT` komutu `WORKDIR` altında yani `/opt/todo` altında çalıştırılacak.

### 4. EXPOSE
EXPOSE komutu ile container'ın hangi portu ya da portları dinlediğini belirtiriz. Portlar default olarak erişime kapalıdır. Container'ı çalıştırırken istersek bu portları erişime açabiliriz.

### 5. ENV
ENV komutu ortam değişkenlerini ayarlamak için kullanılır. Container'i başlatırken isterseniz bu ortam değişkenlerini değiştirebilirsiniz. Bizim örneğimizde `DB_URL` ortam değişkeni `jdbc:h2:mem:tododb` değerini alıyor. Container'da çalışan java application'ın başka bir database'e bağlanması için `docker run` komutu ile container'ı başlatırken bu değişkeni güncelleyebiliriz.

```
docker run \
 -p 9000:9000 \
 -e DB_URL="jdbc:mysql://172.23.0.1:3306/tododb?nullNamePatternMatchesAll=true" \
 ilkinulas/todoapp:1.0
```

### 6. ENTRYPOINT
ENTRYPOINT komutu ile container başlatıldığında hangi komutu çalıştırması gerektiğini belirtiyoruz. Bizim container'ımız `docker run` komutu ile başlatıldığında `java` uygulamamızı çalıştıracak.

## Docker Compose 

Basit bir todo web uygulamasının çalışması için bile iki tane container ayağa kaldırmamız gerekti. Bir tanesi java uygulamamız için, bir tanesi de mysql database'i için. Birden fazla container'dan oluşan Docker uygulamalarını çalıştırmak, bu uygulamalar arasındaki ilişkileri tanımlamak için `docker-compose` aracını kullanabiliriz. 

Docker Compose, YAML formatında yazılan `docker-compose.yml` dosyası ile kontrol edilir. Bu dosyaya container'lar ile ilgili bütün ayarları ekleriz ve `docker-compose up` komutu ile tüm container'ları bir seferde ayağa kaldırabiliriz ve `docker-compose down` komutu ile tüm container'ları durdurabiliriz.

![Docker Compose Logo](/assets/docker/docker_compose_logo.png){: .center-image }

Docker Compose'un başlıca kullanım alanları şunlardır:
 * Development ortamlarının ayarlanmasında kullanılabilir. Geliştirmekte olduğumuz uygulamanın kullandığı database, cache, queue, vb ne varsa `docker-compose.yml` dosyasında tanımlanabilir. Bu sayede geliştirme ortamı basit bir `docker-compose up` komutu ile kullanıma hazır hale getirilebilir.
 * Automated testlerimizde kullanabiliriz. Integration ya da end-to-end testlerin çalışması için test ortamını docker-compose yardımıyla hazırlayıp, testlerimizi çalıştırabiliriz: 
 
```
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```

Aşağıda demo uygulamasını ve mysql database'ini tanımladığım docker-compose.yml dosyasını görebilirsiniz.

<script src="http://gist-it.appspot.com/http://github.com/ilkinulas/kotlin-todo-app/blob/master/docker-compose.yml"></script>

Bu dosyada  `database` ve `app` adında iki tane servis tanımladım. `database` servisi resmi mysql docker imajını kullanıyor, `app` servisi de [`ilkinulas/todoapp`](https://hub.docker.com/r/ilkinulas/todoapp/) docker imajının 1.0 versiyonunu kullanıyor. 

Docker Compose'u çalıştırdığınızda sizin için bir [network](https://docs.docker.com/network/) kurar. `docker network ls` komutuyla kullanılan network'leri listeleyebilirsiniz. Aşağıda app ve database için oluşturulan `kotlintodoapp_default` adındaki network'u görüyorsunuz.

```
$ docker-compose up -d
Creating network "kotlintodoapp_default" with the default driver
Creating kotlintodoapp_app_1      ... done
Creating kotlintodoapp_database_1 ... done
$ docker network ls
NETWORK ID          NAME                    DRIVER              SCOPE
99c09baeb7e4        bridge                  bridge              local
bad5d15f086a        host                    host                local
9dc3598330ba        kotlintodoapp_default   bridge              local
1596fd786600        none                    null                local
```

Aynı docker-compose.yml dosyasında tanımlanan servisler aynı network içerisinde oldukları için birbirlerinı görebilirler. Servis adı üzerinden (IP adresine gerek kalmadan) haberleşebilirler. `todoapp` container'ı için DB_URL ortam değişkeni tanımlarken database servisinin IP'sini değil aşağıdaki gibi adını vermemiz yetiyor:

`jdbc:mysql://database:3306/tododb`

Docker network'u dışından, container içinde çalışan servislere erişmek istersek container'daki internal port'ları external (dışarıdan erişilebilen) port'lar ile eşleştirmemiz gerekir. Aşağıdaki örnekte container'ın 9000 portu, container'ı çalıştıran host'un 9000 portuna bağlanmış. Böylece container içindeki servisin 9000 portuna container dışından da erişilebilir.

```
ports:
    - 9000:9000
```

Docker'a `9000:9000` port eşleşmesini söylemeseydik, docker kullanımda olmayan rastgele bir portu 9000 portu ile eşleştirecekti. Bu durumda container dışından uygulamaya erişebilmek için önce servis verdiği portu bulmamız gerekecekti.

Önce `docker ps` komutu ile container id'sini bulalım. 
```
$ docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                     NAMES
f7064ba67040        mysql:5.7.21            "docker-entrypoint.s…"   8 seconds ago       Up 14 seconds       0.0.0.0:33378->3306/tcp   kotlintodoapp_database_1
ebeb5f4e846d        ilkinulas/todoapp:1.0   "/bin/sh -c 'java -D…"   8 seconds ago       Up 15 seconds       0.0.0.0:33377->9000/tcp   kotlintodoapp_app_1
```

`ilkinulas/todoapp:1.0` image'ını çalıştıran container'ın id'si `ebeb5f4e846d`. Bu container'ın port eşleşmelerini görmek için `docker port` komutunu kullanabiliriz.

```
$ docker port ebeb5f4e846d
9000/tcp -> 0.0.0.0:33377
```

Docker komutlarını çalıştırırken container id'nin hepsini yazmaya gerek yok. Çalışan diğer container'lardan ayırt edilebilecek şekilde ilk birkaç karakterini yazmak yeterli. Örneğin:

```
$ docker port ebe
9000/tcp -> 0.0.0.0:33377
```
Container'ın 9000 portunun host makinedeki 33377 portuna map edildiğini gördükten sonra `http://localhost:33377` adresinden uygulamaya erişebiliriz. Siz kendi bilgisayarınızda bunu denerseniz, docker farklı bir port atayabilir.

Aynı container'dan birden fazla başlatacaksanız host makinenin `hard-coded` portlarını kullanmak yerine docker engine'in bizim için boş portları seçmesi daha kullanışlı olur. 

Docker ve docker-compose araçlarını öğrenmenin en iyi yolu bana sorarsanız bir demo uygulama yapıp onu container içerisinde çalıştırmak. Bu amaçla geliştirdiğim demo uygulamasının [README sayfasında](https://github.com/ilkinulas/kotlin-todo-app) docker imajının build edilmesi ve docker repository'ye publish edilmesi ile ilgili bilgi bulabilirsiniz.

Bir sonraki yazıda artık entegrasyon testleri yazmaya başlıyoruz.