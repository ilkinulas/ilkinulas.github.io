---
layout: post
title: Kotlin Sequences
categories: development kotlin
---
Java 8 ile beraber gelen [Stream](http://www.oracle.com/technetwork/articles/java/ma14-java-se-8-streams-2177646.html)'lerin Kotlin karşılığı  `Sequence`'lardır. Java ile aralarında bir isim çakışması olmasın diye Kotlin'de Sequence adı kullanılmış. 
Collection'lar ve `map`, `filter`, `groupBy` gibi higher-order fonksiyonlar ile veri işlemenin ne kadar kolay olduğunu bir önceki  [yazıda](/development/kotlin/2017/03/10/Kotlin-Collections.html) görmüştük. Bu yazıda da Sequence'lardan yani `lazy Collection`'lardan bahsedeceğiz. Sequence'lar sayesinde java 8 öncesini destekleyen platformlarda (örneğin Android) stream'lerın sağladığı avantajları kullanabilirsiniz. Sequence'ların, stream'lerden farklı olarak, birden fazla CPU'da çalışma destekleri yoktur.

Bu yazıda yer alan örneklerde aşağıdaki `Payment` data sınıfını kullanacağız. Payment sınıfı "free to play" bir oyunun uygulama içi yapılan satın almalarını temsil ediyor. Örneğin [101 Plus](https://play.google.com/store/apps/details?id=net.peakgames.Yuzbir) oyununda kullanıcılar oyun içerisinden kredi kartları ile ödeme yapıp sadece oyun içinde kullanabilecekleri sanal paraları (coins) satın alabilirler. Payment sınıfı satın almanın ne zaman, kim tarafından ve hangi platformdan yapıldığı gibi bilgileri tutuyor.

{% highlight kotlin %}
enum class Platform {
    Android, IOS, Web
}
enum class Currency {
    TRY, USD, EUR
}
data class Payment(
        val date: Long,
        val userId: String,
        val platform: Platform,
        val price: Double,
        val currency: Currency,
        val coins: Long)
{% endhighlight %}

Android platformundan yapılan satın almaların TRY (Türk Lirası) cinsinden toplamını bulmak için filter ve map higher-order fonksiyonlarını kullabiliriz. Doların 3.6 Euro'nun da 3.9 lira olduğunu varsayarsak aşağıdaki örnek bize TRY cinsinden toplamı verir.

{% highlight kotlin %}
val revenueTRY = paymentList.filter { it.platform == Platform.Android }.map {
    when (it.currency) {
        Currency.TRY -> it.price
        Currency.USD -> it.price * 3.6
        Currency.EUR -> it.price * 3.9
    }
}.sum()
{% endhighlight %}

Collection'lar üzerinde çalışan map ve filter fonksiyonları hesaplamalar arasında geçici collection'lar oluştururlar. `paymentList`'e Android platformu filtresi uyguladığımızda içerisinde sadece Android platformundan yapılan satın almaların olduğu ara bir liste oluşur. Daha sonra bu ara listeyi `map` fonksiyonu alır ve map de TRY cinsinden fiyatların olduğu yeni bir ara liste oluşturur. Son olarak da olusan son listenin içindeki fiyatları toplamak için `sum` fonksiyonunu kullanabiliriz.

Sequence'lar, hesaplamalar arasında geçici collection'lar oluşturmadan collection'lar üzerinde çalışabilme imkanı sağlar. Yukarıda yapılan işlemin aynısı Sequence kullanılarak aşağıdaki gibi yapılabilir.

{% highlight kotlin %}
 val revenueTRY = paymentList.asSequence().filter { it.platform == Platform.Android }.map {
    when (it.currency) {
        Currency.TRY -> it.price
        Currency.USD -> it.price * 3.6
        Currency.EUR -> it.price * 3.9
    }
}.sum()
{% endhighlight %}

Yukarıdaki kodun ilk örnekten tek farkı, `filter` ve `map` fonksiyonlarını doğrudan liste üzerinde değil,  `asSequence()` metodu ile oluşturduğumuz bir sequence üzerinde çalıştırmasıdır. 

Kotlin'de Sequence'lar, Collection'lar ile aynı API metodlarına sahip oldukları için Collection'lar ile yapabildiğiniz herşeyi Sequence'lar ile de yapabilirsiniz. Sequence'lar, Collection'ların aksine `lazy` çalışırlar. Sequence'lar üzerinde yapılan işlemler ara işlemler (intermediate) ve sonlandırıcı işlemler (terminal) olmak üzere ikiye ayrılır. Ara işlemler sonuç olarak yeni bir Sequence dönerler. Sonlandırıcı işlemler sonuç olarak bir collection ya da herhangi bir object (sayı, sequence içindeki herhangi bir eleman ...) dönerler. 

Aşağıda `asSequence()` metodu ile oluşturulan bir Sequence'a önce map sonra da filter ara işleminin uygulandığını görüyoruz. Bu örnek kod parçası çalıştırıldığında ekrana hiçbir şey yazmaz. Çünkü sequence'larda ara işlemler `lazy` olarak çalıştırılır.

{% highlight kotlin %}
listOf(1, 2, 3, 4, 5).asSequence()
            .map { print("map ($it) "); it * it }
            .filter { print("filter ($it) "); it > 10 }
{% endhighlight %}

Sequence'ı sonlandırıp bir sonuç elde etmek için `toList()` metodu kullanabiliriz. Sonlandırıcı işlem tüm ara işlemlerin çalıştırılmasını sağlar.

{% highlight kotlin %}
listOf(1, 2, 3, 4, 5).asSequence()
            .map { print("map ($it) "); it * it } // intermediate operation
            .filter { print("filter ($it) "); it > 10 } // intermediate operation
            .toList() // terminal operation

// prints
// map (1) filter (1) map (2) filter (4) map (3) filter (9) map (4) filter (16) map (5) filter (25)            
{% endhighlight %}

## Sequence Oluşturma

Sequence'ları iki farklı şekilde oluşturabiliriz.

* Herhangi bir [Iterable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterable/)'dan  `asSequence()` metodu ile
* [generateSequence](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/generate-sequence.html) standard library fonksiyonu ile

`generateSequence` fonksiyonu iki parametre alıyor. Birinci parametre sequence'in başlangıc elemanı, ikinci parametre ise bir önceki elemanı kullanarak sequence'ta sıradaki elemanı hesaplayan lambda fonksiyonu. Aşağıda Fibonacci sayılarını hesaplayan bir sequence oluşturma örneği görüyorsunuz:

{% highlight kotlin %}
val fibonacci = generateSequence(1 to 1) {
    it.second to it.first + it.second
}.map { it.first }

fibonacci.take(10).forEach { print("$it ") }
//prints 1 1 2 3 5 8 13 21 34 55
{% endhighlight %}

`generateSequence` fonksiyonunun parametre olarak sadece bir generator fonksiyon alan versiyonu da var. Sonsuz sayıda ve rastgele `Payment` üretmek için aşağıdaki generator kullanılabilir:

{% highlight kotlin %}
private val RANDOM = Random()

fun paymentSequence() = generateSequence {
    val date = System.currentTimeMillis() - RANDOM.nextInt(100000)
    val userId = generateSequence { (RANDOM.nextInt(26) + 65).toChar() }.take(6).joinToString("")
    val platform = Platform.values()[RANDOM.nextInt(3)]
    val currency = Currency.values()[RANDOM.nextInt(3)]
    Payment(date, userId, platform, RANDOM.nextDouble(), currency, RANDOM.nextLong())
}
{% endhighlight %}

Payment object'lerinden oluşan sequence'ı ve standard library'deki sequence ve collection metodlarını kullanarak bir kaç örnek hesaplama yapalım. Rastgele oluşturduğumuz Payment'lardan 100 tanesini kullanarak her platform için ayrı ayrı satın alma adetlerini aşağıdaki gibi hesaplayabiliriz.

{% highlight kotlin %}
val numberOfPaymentsByPlatform =
        paymentSequence().take(100)
                .groupBy { it.platform }
                .map { it.key to it.value.count() }
println(numberOfPaymentsByPlatform)

//prints [(Web, 34), (Android, 38), (IOS, 28)]
{% endhighlight %}

Currency'yi hesaba katmadan tüm platformlardan elde edilen geliri aşağıdaki gibi hesaplayabiliriz:

{% highlight kotlin %}
val totalRevenue = paymentSequence().take(100).sumByDouble { it.price }
println(totalRevenue)
{% endhighlight %}

Currency'yi hesaba katmadan platformların ayrı ayrı gelirlerini aşağıdaki gibi hesaplayabiliriz:

{% highlight kotlin %}
val totalRevenueByPlatform = paymentSequence().take(100)
        .groupBy { it.platform }
        .map { it.key to it.value.sumByDouble { it.price } }
println(totalRevenueByPlatform)
{% endhighlight %}

Görüldüğü gibi kullanım şekli bakımından Sequence'ların Collection'lardan bir farkı yok. Eleman sayısı çok fazla olan Collection'lar üzerinde çalışıyorsak Sequence kullanmak geçici (temporary) collection'ların oluşmasını önleyeceği için daha avantajlı olabilir.