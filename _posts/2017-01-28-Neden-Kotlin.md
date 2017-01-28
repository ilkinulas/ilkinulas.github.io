---
layout: post
title: Neden Kotlin?
categories: development kotlin
---
![kotlin logo](/assets/neden_kotlin/logo_Kotlin.png){: .center-image }

## Neden olmasin?
[JetBrains](http://www.jetbrains.com/) 6 yil once Kotlin programlama dili uzerinde calismaya baslamis ve **Kotlin 1.0** 'in [duyurulmasinin](https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/) uzerinden 1 yil gecmis. Bu yaziyi Kotlin'i henuz kullanmamis Java developer'lari Kotlin'i en azindan bir kere denemeleri icin kaleme aliyorum. Yeni bir programlama dili ogrenmenin kime ne zarari olabilir ki? Kotlin ogrenmenin soyle bir yan etkisi olabiliyor bastan soyleyeyim : Bir daha Java ile kod yazmak istemeyebilirsiniz.

2002 yilindan beri profesyonel olarak, cok sevdigim Java programla dili ile uygulamalar gelistiriyorum. Java ile web uygulamalari, [telekom sunuculari](https://en.wikipedia.org/wiki/Online_charging_system) ve [mobil oyunlar](http://www.peakgames.net) gelistirmis birisi olarak keske Java programlama diline su ozelligi de ekleseler dedigim herseyi Kotlin'de buldugumu soyleyebilirim. Birazdan bunlardan kisa kisa bahsedecegim. 

Java Sanal Makinesi (JVM), [java bytecode](https://en.wikipedia.org/wiki/Java_bytecode)'larini hafizaya yukleyip calistirabilen, cogunlukla C++ ile yazilmis devasa bir programdir. Eger derleyiciniz programlama dilinizi alip JVM'in anlayacagi bytecode'a cevirebilirse yazdiginiz programlar sanal makine uzerinde calisabilir. Peki neden JVM'e ihtiyac duyariz? JVM bize sunlari sunar:

 * __Garbage Collection__ : Dinamik hafiza yonetimi sayesinde kullanilmayan instance'lar, yani copler, garge collector tarafindan temizlenir. 
 * __Write Once Run (almost) Every Where__ : Her platform icin farkli bir sanal makine oldugu icin bytecode'a cevrilen programiniz Windows, Mac, Linux farketmeksizin ayni sekilde calisacaktir.
 * __Performance & Robustness__ : JVM'in (reference implementation) 20 yillik bir gecmisi var ve yillardir finans, telekom, online alisveris, oyun gibi bircok alanda kullaniliyor. Uygulamanizin karakteristigine gore farkli JVM'ler kullanabilirsiniz. 
    * [HotSpot](https://en.wikipedia.org/wiki/HotSpot), Reference Java VM implementation
    * [Open JDK](https://en.wikipedia.org/wiki/OpenJDK), free & open JVM implementation
    * [Realtime JVMs](https://en.wikipedia.org/wiki/Real_time_Java)
 * __Monitoring tools__ : Sanal makinenin durumunu izlemek icin ve daha performansli calismasini saglamak icin bir cok arac mevcut. [VisualVM](http://visualvm.java.net/), [JProfiler](http://www.ej-technologies.com/products/jprofiler/overview.html), [GCViewer](http://www.tagtraum.com/gcviewer.html), [jmap](http://docs.oracle.com/javase/7/docs/technotes/tools//share/jmap.html), [jstack](http://docs.oracle.com/javase/7/docs/technotes/tools//share/jstack.html) bunlardan sadece bazilari.

 JVM'in bedavadan yukaridaki ozellikleri sunmasi JVM uzerinde calisabilen Java'dan baska bircok dilin dogmasina yol acmistir. Popupler JVM dillerinden bazilari : Kotlin, Clojure, Scala, JRuby, Jython, Groovy, LuaJ.

Simdi tekrar Kotlin'e geri donelim. 

### 1. Statically Typed

Kotlin statically typed bir programlama dilidir. Kotlin compiler (kotlinc) kotlin ile yazdiginiz kaynak kodlarini java byte code'a cevirir. Tipler derleme sirasinda belirlidir. Ornegin bir degiskenin tipi String ise o degiskene Integer atayamazsiniz, compiler buna izin vermez. Static (Java, C#, C++) ve dinamik (Javascript, Python) programlama dilleri ile calismis birisi olarak kisisel tercihim statik diller. Cunku :

  * Static dillerde compiler'lar, kodunuzdaki hatalari kodunuz daha calismadan (production'a gitmeden) soyleyebilir.
  * IDE'ler tipleri bildikleri icin size daha gelismis oneriler sunabilir. Cok gelismis refactoring destegi saglayabilirler.

### 2. Kotlin, Java ile Uyumlu Calisabilir (Interoperable)

Kotlin'den Java'yi ve Java'dan Kotlin'i cagirmak mumkun. Bu iki farkli dunya arasindaki haberlesme o kadar puruzsuz ki kullandiginiz library'nin Java ile mi yoksa Kotlin ile mi yazildigini anlamaniz mumkun degil. Iki dilin uyumu, mevcut java projelerimizin yeni ozelliklerini Kotlin ile gelistirme imkani veriyor. 

Java cok populer bir programala dili oldugu icin her ihtiyaca gore yazilmis kutuphaneler bulmak cok kolay (3rd party jars). Kotlin ile yazdiginiz kodlar bu kutuphanelerin hepsini kullanabilir. Ornegin


{% highlight java %}
import org.apache.commons.io.FileUtils
import java.io.File

...

  val file = File("project.properties")
  val fileContent = FileUtils.readLines(file, "UTF-8")
  println(fileContent)

...

{% endhighlight %}

### 3. Kotlin ile Guvendesiniz. Elveda NPE!
Kotlin'de __Nullable__ ve __Non-Null__ Type sistemin bir parcasidir. Bu sayede Kotlin compiler null olabilecek referanslar ile asla null olamayacak referanslari ayirt edebilir.

{% highlight java %}
var s:String = null // Hata !!
//Compiler der ki : null can not be a value of a non-null type String
{% endhighlight %}

Bir degiskene null referans atamak isterseniz tipin sonuna __?__ (soru isareti) eklemeniz gerekir. Ornegin __String?__ nullable string demektir ve Kotlin icin __String__ ve __String?__ birbirinden farkli iki tiptir. 

{% highlight java %}
var s:String="test"
var s1:String? = null
s=s1 // Hata !!
//Compiler der ki: error: type mismatch: inferred type is String? but String was expected
{% endhighlight %}


Kotlin official dokumantasyonunda null reference'tan [Billion dolar mistake](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions) olarak bahseder. Kotlin type sistemi daha detayli bir yaziyi hakediyor. Bu yazida daha fazla detaya girmeyecegim.

### 4. Kotlin ile Daha Az Satir Kod

15 yillik yazilim muhendisligi tecrubem bana sunu ogretti: Yazdigimiz kodlarin satir sayisi ile projemizdeki hata (bug) sayisi arasinda dogru oranti var. Yani ne kadar cok kod o kadar cok hata. 

Kotlin ile ayni seyleri Java'ya gore daha az satir kod yazarak ifade edebiliriz. Asagidaki ozellikleri sayesinde Kotlin, Java'ya gore daha oz (concise) bir dildir.

* Type inference
* Extension method
* Lambda function
* Data class
* Operator overloading

Kotlin bu ozelliklerin hic birisini sifirdan icat etmedi. Java programcilarinin yillardir gipta ile baktigi dillerdeki iyi ozellikleri JVM'e tasidi. Bu bloga yukaridaki maddelerin herbiri icin ayri birer yazi ekleyecegim icin simdilik bunlar ile ilgili detaya girmiyorum.

### 5. Kotlin ile Android Uygulamasi Yazabiliriz

Kotlin compiler Java 6 uyumlu bytecode uretir. Bu sayede Kotlin ile Android uygulamalari gelistirmek mumkun. (PeakGames'te java ile gelistirdigimiz oyunlara yeni ozellikler eklerken Kotlin kullaniyoruz.)

### 6. IDE destegi

JetBrains tarafindan gelistirilen bir programlama dili oldugu icin __IntelliJ Idea__ Kotlin destegi ile beraber geliyor. Ayrica Eclipse'ten vazgecemeyenler icin de [Eclipse plugin](https://marketplace.eclipse.org/content/kotlin-plugin-eclipse) mevcut. 

##  Kotlin'i nasil ogrenebilirim?
Buraya kadar okuduysaniz Kotlin'i denemeye hazirsiniz demektir. Java bilen bir yazilimci icin Kotlin ogrenmek gercekten cok kolay.
Kotlin ogrenmeye baslamak icin JetBrains tarafindan hazirlanan [referans dokumanlari](https://kotlinlang.org/docs/reference/) bence yeterli. 

Okuyarak degil de uygulayarak daha iyi ogreniyorum diyenler icin de [Kotlin Koans](https://kotlinlang.org/docs/tutorials/koans.html) var. 
Kotlin Koans, icinde bir cok gecmeyen (failing) unit test olan bir [proje](https://github.com/Kotlin/kotlin-koans). Buradaki unit testleri gecer (yesil) hale getirmeye calisarak da Kotlin ogrenebilirsiniz. [Bu](https://github.com/ilkinulas/kotlin-koans) github repo'sunda benim Kotlin-Koans cozumlerimi bulabilirsiniz.
