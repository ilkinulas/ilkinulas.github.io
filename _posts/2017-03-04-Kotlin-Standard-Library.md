---
layout: post
title: Kotlin Standard Library
categories: development kotlin
excerpt_separator: <!--more-->
---
`Kotlin Standard Library`, Kotlin ile kod yazarken sürekli başvurulan bir araç kutusu olarak düşünülebilir. Standard Library ile ilgili dokümanlara [buradan](https://kotlinlang.org/api/latest/jvm/stdlib/), kaynak koduna da [buradan](https://github.com/JetBrains/kotlin/tree/master/libraries/stdlib) ulaşabilirsiniz. JDK üzerine inşa edilmiş ve JDK'nın Kotlin ile kullanılmasını sağlayan Standard Library içerisinde:

- Collection ve Sequence sınıflarının kullanımını kolaylaştıran [extension metodları](development/kotlin/2017/02/11/Kotlin-Extensions.html)
- String, char dizileri ve regular expression'lar için extension'lar ve yardımcı sınıflar
- JDK içindeki  `IO`, `threading` ve dosya sistemi ile ilgili sınıfların Kotlin'den rahatça kullanılmasını sağlayan extension'lar ve yardımcı sınıflar
- Bu yazıda üzerinde duracağımız, Kotlin'e özgü özel kalıplar yazmamızı sağlayan `let` `apply` `with` `use`, `synchronized` gibi `higher-order function`lar vardır.

<!--more-->

### Higher-Order Functions
`Higher-order` fonksiyonlar, kısaca HOF, bir ya da birden fazla fonksiyonu parametere olarak alabilen ve/veya sonuç olarak başka bir fonksiyonu dönebilen fonksiyonlardır.

{% highlight java %}
fun greaterThan(n: Int): (Int) -> Boolean {
    return fun(x: Int) = x > n
}

fun <T> filter(list: List<T>, predicate: (T) -> Boolean): List<T> {
    val destination = ArrayList<T>()
    for (item in list) if (predicate(item)) destination.add(item)
    return destination
}

fun main(args: Array<String>) {
    val greaterThan5 = greaterThan(5)

    greaterThan5(4) // false
    greaterThan5(6) // true

    filter(listOf(1, 2, 3, 4, 5, 6, 7), greaterThan5) // [6, 7]
}
{% endhighlight %}

Yukarıdaki `greaterThan` ve `filter` fonksiyonları HOF örnekleridir. 

- `greaterThan` `(Int) -> Boolean` tipinde başka bir fonksiyon döner
- `filter` parametre olarak `(T) -> Boolean` tipinde bir fonksiyon alır.


Kotlin standard library ile gelen bazı higher-order fonksiyonlardan bahsedeceğim. Kotlin ile ilk tanıştığım zamanlarda bu fonksiyonları anlamak ve alışmak epey zamanımı almıştı. Bu yüzden bu önemli fonksiyonlar için kullanım örnekleri de içeren Türkçe bir kaynak hazırlamak istedim.

# TODO ()

{% highlight java %}
fun TODO(): Nothing = throw NotImplementedError()
{% endhighlight %}

`TODO` fonksiyonu ile bir özelliğin henüz kullanıma hazır olmadığını belirtebiliriz. 

{% highlight java %}
fun vipFeature() {
    TODO("VIP")
}
{% endhighlight %}
Bitmemiş bu özellik kullanılmak istenirse runtime'da `kotlin.NotImplementedError` fırlatılır.
<pre>
Exception in thread "main" kotlin.NotImplementedError: An operation is not implemented: VIP
	at stdlib.StdlibTestsKt.vipFeature(StdlibTests.kt:21)
	at stdlib.StdlibTestsKt.main(StdlibTests.kt:32)
</pre>


# apply ()
{% highlight java %}
fun <T> T.apply(f: T.() -> Unit): T { f(); return this }
{% endhighlight %}

`apply` bütün Kotlin tipleri ile kullanılabilen bir extension'dır. 
[Bu yazımda](development/kotlin/2017/02/24/Kotlin-Instance-Olusturma.html) `builder pattern`'den bahsederken `function type with receiver` kavramından bahsetmiştim. `apply` extension metodu, uygulandığı instance'ı receiver olarak alan bir lambda expression çalıştırır ve sonuç olarak aynı instance'ı geri döner. 

Bir instance üzerinde değişiklikler yapıp sonra aynı instance'ı geri dönmek istediğimizde kullanılabilir. Örneğin:

{% highlight java %}
//Kotlin
Socket().apply {
    soTimeout = 10000
    keepAlive = true
    tcpNoDelay = false
}.connect(InetSocketAddress(host, port))
{% endhighlight %}

İstenilen adrese (host ve port) istenilen özelliklere sahip bir socket bağlantısı kurmak için kullanılan yukarıdaki kodun Java karşılığı aşağıdaki gibidir:

{% highlight java %}
//Java
try {
    Socket socket = new Socket();
    socket.setKeepAlive(true);
    socket.setSoTimeout(10000);
    socket.setTcpNoDelay(false);
    socket.connect(new InetSocketAddress(host, port));
} catch (IOException e) {
    e.printStackTrace();
}
{% endhighlight %}

# let () 
{% highlight java %}
fun <T, R> T.let(f: (T) -> R): R = f(this)
{% endhighlight %}

`let`, `apply`'a benzer fakat uygulandığı instance'ı değil instance'ı parametre alan lambda'nın sonucunu döner. Java'daki `if (myObject != null)` ifadesinin Kotlin'deki karşılığı `let` olabilir. 

{% highlight java %}
class User(val id: String, val name: String, val age: Int)

val age = db.findUser("123")?.let {
    println(it.name)
    it.age
}
{% endhighlight %} 

Yukarıdaki örnekte  `let` ile ilgili 3 özellik gözüküyor:
* `let` sayesinde `db.findUser` metodundan dönen user objesinin erişilebilirliği (scope) daraltılmış oldu (ki bu iyi birşey). `let` bloğundan sonra user instance'ı scope'tan çıkar.
* `let` bloğu içinde user objesine `it` üzerinden erişebiliriz.
* `apply`'dan farklı olarak `let` user instance'ı değil bloğun sonucunu döner. Bizim örneğimizde user'ın yaşını dönüyor.

# with ()

{% highlight java %}
fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
{% endhighlight %}

`with()` herhangi bir tipe ait olmayan `top-level` bir fonksiyondur. Aşağıdaki örnekteki gibi bir instance'ın farklı metodlarını peş peşe çağırdığımız zaman instance'ın değişken adını tekrar tekrar yazmak istemediğimizde kullanılabilir.

{% highlight java %}
val dialog = Dialog()
with(dialog) {
    title = "Do you want to save the changes?"
    body = "Your changes will be lost if you don't save them."
    titleColor = Color.RED
    bodyColor = Color.BLACK
}
{% endhighlight %}  

# lazy ()
`lazy` metodu sayesinde masraflı bir metodun çağırılmasını, ilk ihtiyaç duyulan ana kadar erteleyebiliriz. 

{% highlight java %}
val lazyUser = lazy { db.findUser("2") }
//database'e henuz gidilmedi

println(lazyUser.value?.name)
//database'e gidildi
{% endhighlight %}  

Yukarıdaki örnekte `db.findUser("2")` lazyUser'ın tanımlandığı satırda işletilmez, lazyUser'ın `value` alanına ilk erişimde işletilir. Peş peşe çağırıldığında her seferinde database'de user aramaz. İlk bulduğu user'ı döner. 

`lazy` bloğu `Thread-Safe`'tir. Birden fazla thread aynı anda lazy bloğunu işletse bile tek bir sonuç üretilir.

# use()

Java'daki [try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html`)'ın Kotlin karşılığı standard library'deki `use` fonksiyonudur. 
Kotlin compiler'ın ürettigi byte code Java 6 desteklediği için `use`, [AutoCloseable](https://docs.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html) interface'inin değil [Closeable](https://docs.oracle.com/javase/7/docs/api/java/io/Closeable.html) interface'inin bir extension metodur.

Aşağıdaki örnek `project.properties` dosyasını açıyor, okuyor ve işi bitince otomatik olarak kapatıyor:

{% highlight java %}
val projectProperties = Properties()
Files.newInputStream(Paths.get("project.properties")).use { inputStream ->
    projectProperties.load(inputStream)
}
{% endhighlight %}  


# synchronized()

Kotlin'de Java'daki `synchronized` keyword'u yoktur. Java'daki `synchronized` bloklarının yerine Kotlin'de  standard library'deki `synchronized()` fonksiyonu kullanılabilir.

{% highlight java %}
val lock = Object()

synchronized(lock, { /* thread-safe block */ })

synchronized(lock) {

    //thread-safe blok

}
{% endhighlight %}  

Kotlin standard library içinde yer alan bazı Higher-order fonksiyonlardan bahsettik. Bir sonraki yazıda Kotlin'de Collection'ları ve Sequence'ları yazacağım.