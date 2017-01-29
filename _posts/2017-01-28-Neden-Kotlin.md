---
layout: post
title: Neden Kotlin?
categories: development kotlin
---

## Neden olmasın?
[JetBrains](http://www.jetbrains.com/) 6 yıl önce Kotlin programlama dili üzerinde calismaya başlamış ve **Kotlin 1.0** 'ın [duyurulmasının](https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/) üzerinden 1 yıl geçmiş. Bu yazıyı Kotlin'i henüz denememiş olan Java yazılımcılarının Kotlin'i en azından bir kere denemeleri için kaleme alıyorum. Yeni bir programlama dili öğrenmenin kime ne zararı olabilir ki? Kotlin öğrenmenin şöyle bir yan etkisi olabiliyor baştan söyleyeyim : Bir daha Java ile kod yazmak istemeyebilirsiniz.

2002 yılından beri profesyonel olarak, çok sevdiğim Java programla dili ile uygulamalar geliştiriyorum. Java ile web uygulamaları, [telekom sunucuları](https://en.wikipedia.org/wiki/Online_charging_system) ve [mobil oyunlar](http://www.peakgames.net) geliştirmiş birisi olarak keşke Java programlama diline şu özelliği de ekleseler dediğim herşeyi Kotlin'de bulduğumu söyleyebilirim. Birazdan bunlardan kısa kısa bahsedeceğim. 

Java Sanal Makinesi (JVM), [java bytecode](https://en.wikipedia.org/wiki/Java_bytecode)'larını hafızaya yükleyip çalıştırabilen, çoğunlukla C++ ile yazılmış devasa bir programdır. Eğer derleyiciniz programlama dilinizi alıp JVM'in anlayacağı bytecode'a çevirebilirse yazdığınız programlar sanal makine üzerinde çalışabilir. Peki neden JVM'e ihtiyaç duyarız? JVM bize şunları sunar:

 * __Garbage Collection__ : Dinamik hafıza yönetimi sayesinde kullanılmayan instance'lar, yani çöpler, garbage collector tarafından temizlenir. 
 * __Write Once Run (almost) Every Where__ : Her platform için farklı bir sanal makine olduğu için bytecode'a çevrilen programınız Windows, Mac, Linux farketmeksizin aynı şekilde calışacaktır.
 * __Performance & Robustness__ : JVM'in (reference implementation) 20 yıllık bir geçmişi var ve yıllardır finans, telekom, online alışveriş, oyun gibi bircok alanda kullanılıyor. Uygulamanızın karakteristiğine göre farklı JVM'ler kullanabilirsiniz. 
    * [HotSpot](https://en.wikipedia.org/wiki/HotSpot), Reference Java VM implementation
    * [Open JDK](https://en.wikipedia.org/wiki/OpenJDK), free & open JVM implementation
    * [Realtime JVMs](https://en.wikipedia.org/wiki/Real_time_Java)
 * __Monitoring tools__ : Sanal makinenin durumunu izlemek için ve daha performanslı çalısmasını sağlamak için birçok araç mevcut. [VisualVM](http://visualvm.java.net/), [JProfiler](http://www.ej-technologies.com/products/jprofiler/overview.html), [GCViewer](http://www.tagtraum.com/gcviewer.html), [jmap](http://docs.oracle.com/javase/7/docs/technotes/tools//share/jmap.html), [jstack](http://docs.oracle.com/javase/7/docs/technotes/tools//share/jstack.html) bunlardan sadece bazıları.

 JVM'in bedavadan yukarıdaki özellikleri sunması JVM üzerinde calisabilen Java'dan baska birçok dilin doğmasına yol açmıştır. Popupler JVM dillerinden bazıları : Kotlin, Clojure, Scala, JRuby, Jython, Groovy, LuaJ.


![kotlin logo](/assets/neden_kotlin/logo_Kotlin.png){: .center-image }

Şimdi tekrar Kotlin'e geri dönelim. Kotlin'in öğrenmeye değer bir programlama dili olduğunu düşünüyorum çünkü;

### 1. Statically Typed

Kotlin, _statically typed_ bir programlama dilidir. Kotlin compiler (kotlinc) kotlin ile yazdığınız kaynak kodlarını java byte code'a çevirir. Tipler derleme sırasında belirlidir. Ornegin bir değişkenin tipi String ise o değişkene Integer atayamazsınız, compiler buna izin vermez. Static (Java, C#, C++) ve dinamik (Javascript, Python) programlama dilleri ile calışmış birisi olarak kişisel tercihim statik diller. Çünkü :

  * Static dillerde compiler'lar, kodunuzdaki hataları kodunuz daha çalişmadan (production'a gitmeden) söyleyebilir.
  * IDE'ler tipleri bildikleri için size daha gelişmiş öneriler sunabilir. Çok gelişmiş refactoring desteği sağlayabilirler.

### 2. Kotlin, Java ile Uyumlu Çalışabilir (Interoperable)

Kotlin'den Java'yı ve Java'dan Kotlin'i çağırmak mumkun. Bu iki farklı dunya arasindaki haberlesme o kadar pürüzsüz ki kullandığınız library'nin Java ile mi yoksa Kotlin ile mi yazıldığını anlamanız mümkün değil. İki dilin uyumu, mevcut java projelerimizin yeni ozelliklerini Kotlin ile geliştirme imkanı veriyor. 

Java çok popüler bir programala dili olduğu için her ihtiyaca göre yazılmış kütüphaneler bulmak çok kolay (3rd party jars). Kotlin ile yazdığınız kodlar bu kütüphaneleınr hepsini kullanabilir. Örneğin:


{% highlight java %}
import org.apache.commons.io.FileUtils
import java.io.File

...

  val file = File("project.properties")
  val fileContent = FileUtils.readLines(file, "UTF-8")
  println(fileContent)

...

{% endhighlight %}

### 3. Kotlin ile Güvendesiniz. Elveda NPE!
Kotlin'de __Nullable__ ve __Non-Null__ Type sistemin bir parçasıdır. Bu sayede Kotlin compiler null olabilecek referanslar ile asla null olamayacak referansları ayırt edebilir.

{% highlight java %}
var s:String = null // Hata !!
//Compiler der ki : null can not be a value of a non-null type String
{% endhighlight %}

Bir değişkene null referans atamak isterseniz tipin sonuna __?__ (soru işareti) eklemeniz gerekir. Örneğin __String?__ nullable string demektir ve Kotlin icin __String__ ve __String?__ birbirinden farklı iki tiptir. 

{% highlight java %}
var s:String="test"
var s1:String? = null
s=s1 // Hata !!
//Compiler der ki: error: type mismatch: inferred type is String? but String was expected
{% endhighlight %}


Kotlin official dokümantasyonunda null reference'tan [Billion dolar mistake](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions) olarak bahsediliyor. Kotlin type sistemi daha detaylı bir yazıyı hakediyor. Bu yazıda daha fazla detaya girmeyeceğim.

### 4. Kotlin ile Daha Az Satır Kod

Yazılım projelerinin büyüklüğü (kodların satır sayısı) ile projelerdeki hata (bug) sayisi arasinda dogru oranti olduğunu düşünüyorum. Yani ne kadar çok kod o kadar çok hata. 

Kotlin ile aynı şeyleri Java'ya göre daha az satır kod yazarak ifade edebiliriz. Aşağidaki özellikleri sayesinde Kotlin, Java'ya göre daha öz (concise) bir dildir.

* Type inference
* Extension method
* Lambda function
* Data class
* Operator overloading

Kotlin bu özelliklerin hiç birisini sıfırdan icat etmedi. Java programcılarının yıllardır gıpta ile baktığı dillerdeki iyi özellikleri JVM'e taşıdı. Bu blog'a yukarıdaki maddelerin herbiri için ayrı birer yazı ekleyeceğim için şimdilik bunlar ile ilgili detaya girmiyorum.

### 5. Kotlin ile Android Uygulaması Yazabiliriz

Kotlin compiler Java 6 uyumlu bytecode üretir. Bu sayede Kotlin ile Android uygulamaları geliştirmek mümkün. (PeakGames'te java ile geliştirdiğimiz oyunlara yeni özellikler eklerken Kotlin kullanıyoruz.)

Kotlin'in Android ile ilgili planlarina [bu linkten](https://blog.jetbrains.com/kotlin/2016/03/kotlins-android-roadmap/) ulaşabilirsiniz. Özetle:

  * Kotlin standart library'nin metod sayısını azaltmaya çalışıyorlar. Bildiğiniz gibi Android uygulamalarında (multi dex kullanmıyorsanız) 64K metod sayısı limiti var. Bu yüzden APK'nin içine girecek kütüphanelerin metod sayısı onemli. Ben bu yazıyı hazirlarken metod sayısı toplam 7191'di : Kotlin Standart Libarary 6289, Kotlin Runtime 902.  <a href="http://www.methodscount.com/?lib=org.jetbrains.kotlin%3Akotlin-stdlib%3A1.0.0"><img src="https://img.shields.io/badge/Methods and size-core: 6289 | deps: 902 | 636 KB-e91e63.svg"/></a>

  * Gradle build'lerinde _incremental compilation_ destekleniyor. Bu sayede sadece değişen sınıflar ve değişen sınıfları kullanan sınıflar derlendiği için build süreleri kısalıyor.

  * [Jack & Jill](http://tools.android.com/tech-docs/jackandjill) ve Instant Run özellikleri henüz tam anlamıyla desteklenmiyor. Fakat roadmap'te var.

### 6. IDE Desteği

JetBrains tarafından geliştirilen bir programlama dili olduğu için __IntelliJ Idea__ Kotlin desteği ile beraber geliyor. Ayrıca Eclipse'ten vazgeçemeyenler için de [Eclipse plugin](https://marketplace.eclipse.org/content/kotlin-plugin-eclipse) mevcut. 

##  Kotlin'i Nasıl Öğrenebilirim?
Buraya kadar okuduysanız Kotlin'i denemeye hazırsınız demektir. Java bilen bir yazılımcı için Kotlin öğrenmek gerçekten cok kolay.
Kotlin öğrenmeye başlamak için JetBrains tarafından hazırlanan [referans dokümanlari](https://kotlinlang.org/docs/reference/) bence yeterli. 

Okuyarak değil de uygulayarak daha iyi öğreniyorum diyenler için de [Kotlin Koans](https://kotlinlang.org/docs/tutorials/koans.html) var. 
Kotlin Koans, içinde birçok geçmeyen (failing) unit test olan bir [open source proje](https://github.com/Kotlin/kotlin-koans). Buradaki unit testleri geçer (yeşil) hale getirmeye çalışarak da Kotlin öğrenebilirsiniz. [Bu](https://github.com/ilkinulas/kotlin-koans) github repo'sunda benim Kotlin-Koans çözümlerim var.

2017'ye yeni bir programlama dili öğrenerek girmek isteyenler için Kotlin bence çok iyi bir seçim olur.
