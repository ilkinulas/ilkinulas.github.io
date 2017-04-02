---
layout: post
title: Kotlin'de Delegasyon
categories: development kotlin
excerpt_separator: <!--more-->
---
Bir class'ı extend ettiğimizde sub-class ve super-class arasında kırılgan bir bağımlılık oluşur. Bu bağımlılığın kırılgan olmasının sebebi super-class'ta yapılan değişikliklerin sub-class'ların çalışmasını bozabilme riskidir. Inheritance'ın bu yan etkisi yüzünden Kotlin'de class'lar `final`'dır (yani extend edilemezler). Eğer bir sınıfı genişletilebilir (extendable) yapmak istiyorsanız sınıfı `open` olarak tanımlamanız gerekir. `open` sınıflarda değişiklik yaparken bu değişikliklerden sub-sınıfların da etkilenebileceğini aklımızdan çıkartmamamız lazım.

<!--more-->

## Class Delegation

Bazen `open` olmayan, yani extend etmeyi düşünmediğimiz sınıflara yeni özellikler eklemek isteyebiliriz. Böyle durumlarda `Decorator` tasarım kalıbı uygulanabilir. Decorator tasarım kalıbı şöyle gerçeklenebilir:

* Yeni özellik eklemek istediğimiz orjinal sınıf ile aynı `interface`'e sahip yeni bir sınıf oluşturulur.
* Orjinal sınıfın bir instance'ı yeni sınıfa bir `field` olarak eklenir.
* Değişmesini istediğimiz interface metodu dışındakı tüm metodlar orjinal sınıf instance'ına paslanır.

Aşağıda FilteringList adında bir `MutableList` implementation'ı var. Bu listeye sadece bizim istediğimiz filtreye uygun elemanların girmesini istiyoruz. `addXXX` metodlarını değiştirerek istediğimiz yeni özelliği ekleyebiliriz. Fakat MutableList interface'inde bulunan diğer metodlar için de orjinal listenin ilgili metodunu çağıran implementasyonlar eklememiz gerekiyor.

{% highlight kotlin %}
class FilteringList<T>(val filter: (T) -> Boolean) : MutableList<T> {
    private val original = ArrayList<T>()

    override val size
        get() = original.size

    override fun add(element: T): Boolean {
        if (!filter(element)) return false
        return original.add(element)
    }

    override fun add(index: Int, element: T) {
        if (filter(element)) original.add(index, element)
    }

    override fun addAll(index: Int, elements: Collection<T>): Boolean = original.addAll(index, elements.filter(filter))

    override fun addAll(elements: Collection<T>): Boolean = original.addAll(elements.filter(filter))

    //Boilerplate code alarm !!!
    override fun contains(element: T): Boolean = original.contains(element)
    override fun containsAll(elements: Collection<T>): Boolean = original.containsAll(elements)
    override fun get(index: Int): T = original[index]
    override fun indexOf(element: T): Int = original.indexOf(element)
    override fun isEmpty(): Boolean = original.isEmpty()
    override fun iterator(): MutableIterator<T> = original.iterator()
    override fun lastIndexOf(element: T): Int = original.lastIndexOf(element)
    override fun clear() = original.clear()
    override fun listIterator(): MutableListIterator<T> = original.listIterator()
    override fun listIterator(index: Int): MutableListIterator<T> = original.listIterator(index)
    override fun remove(element: T): Boolean = original.remove(element)
    override fun removeAll(elements: Collection<T>): Boolean = original.removeAll(elements)
    override fun removeAt(index: Int): T = original.removeAt(index)
    override fun retainAll(elements: Collection<T>): Boolean = original.retainAll(elements)
    override fun set(index: Int, element: T): T = original.set(index, element)
    override fun subList(fromIndex: Int, toIndex: Int): MutableList<T> = original.subList(fromIndex, toIndex)
}
{% endhighlight %}

Yukarıdaki örnek Kotlin'de yazılmış olmasına rağmen bir sürü `boilerplate` kod var. Neyse ki Kotlin compiler bizim için bu laf kalabalığı kodu otomatik olarak üretebiliyor. Bunun için class delagation kullanmamız gerekiyor. Kotlin'de bir interface'i implement ettiginizde, bu interface'in implementasyonunu `by` keyword'u ile başka bir sınıfa `delegate` edebilirsiniz.

Class delegation kullanıldığında FilteringList aşağıdaki gibi `boilerplate` kodlardan arınmış oluyor.

{% highlight kotlin %}
class FilteringList<T>(val original: ArrayList<T>, val filter: (T) -> Boolean)
    : MutableList<T> by original {

    override val size
        get() = original.size

    override fun add(element: T): Boolean {
        if (!filter(element)) return false
        return original.add(element)
    }

    override fun add(index: Int, element: T) {
        if (filter(element)) original.add(index, element)
    }

    override fun addAll(index: Int, elements: Collection<T>): Boolean = 
            original.addAll(index, elements.filter(filter))
    override fun addAll(elements: Collection<T>): Boolean = 
            original.addAll(elements.filter(filter))
}
{% endhighlight %}

FilteringList sınıfını aşağıdaki gibi kullanabiliriz:

{% highlight kotlin %}
val evenNumbers = FilteringList(ArrayList(), { x: Int -> x % 2 == 0 })
for (i in 1..10) {
    evenNumbers.add(i)
}
evenNumbers.forEach { print(" $it") }
{% endhighlight %}
 

## Delagated Properties

Kotlin programla dilinde `var` ile mutable `val` ile immutable class property'leri tanımlayabiliriz. Property'leri tanımlarken optional olarak `initializer`, `getter` ve `setter` kullanılabilir.

{% highlight kotlin %}
class Date {
    var dayOfWeek = 1 // initializer
        get() = field // default getter'i explicit olarak yazmaya gerek yok
        set(value) {
            if (value in 1..7) field = value
    }
}
{% endhighlight %}

Property getter ve setter metodları karmaşık logic'ler içerebilir ve başka sınıflarda ve başka property'lerde yeniden kullanmak isteyebiliriz. Örneğin bir field'ın `lazy` bir şekilde (getter ilk kez çağırıldığında) oluşturulması ya da bir field'in değeri değiştirildiginde (setter)  istediğimiz bir kod bloğunun çalıştırılması gibi karmaşık işlemleri `Delegated Properties` olarak yazıp `reuse` edebiliriz. 

Örneğin aşağıdaki kod parçası name property'sinin getter ve setter işlemlerinin `LoggingDelegate` adında bir sınıf tarafından yapılacağını gösteriyor.

{% highlight kotlin %}
class User {
    var name : String by LoggingDelegate("")
}
{% endhighlight %}

LoggingDelegate sınıfını Kotlin convention'larına göre aşağıdaki gibi yazabiliriz. Bu delegate sınıfı bir property'nin get/set metodları çağırıldığında işlemi konsola yazıyor:

{% highlight kotlin %}
class LoggingDelegate<T>(initial: T) {
    private var value = initial

    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        println("${property.name} property of $thisRef is requested")
        return value
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        println("Updating ${property.name} of $thisRef to $value")
        this.value = value
    }
}
{% endhighlight %}

Kotlin standard library'de  yukarıda örnek olarak verilen LoggingDelegate'ten daha kullanışlı Delegate sınıfları var. 

### Delegates.vetoable
Property'nin değerini belirli bir kurala göre değiştirmek istediğimizde bu delegate'i kullanabiliriz. Örneğin User sınıfının `age` property'si sadece pozitif tam sayılar alabiliyorsa aşağıdaki gibi bir `vetoable` delegate kullanılabilir.

{% highlight kotlin %}
class User {
    var age: Int by Delegates.vetoable(0) { property, old, new ->
        new > 0
    }
}

val user = User()
user.age = 37
user.age = -5
println(user.age) // prints 37
{% endhighlight %}

### Delegates.observable

Bir property'nin değişimlerinden haberdar olmak istiyorsak `observable` delegate kullanabiliriz. Örneğin:

{% highlight kotlin %}
data class Location(val lat: Double, val long: Double)

class User {
    var location: Location by Delegates.observable(Location(0.0, 0.0)) { property, old, new ->
        println("User location changed from $old to $new")
    }
}
{% endhighlight %}

### lazy
Bazen bir propery'nin değerini hesaplamak masraflı bir işlem olabilir. Böyle durumlarda property değerinin ilk erişimde hesaplanması mantıklı olur. Böyle durumlarda `lazy delegation` kullanılabilir. Aşağıdaki örnekte kullanıcının tüm satın almaları `allPayments` lazy property'sine ilk kez erişildiğinde hesaplanıyor. Daha sonraki erişimlerde daha önce hesaplanan payment listesi kullanılır, database'e tekrar gidilmez.

{% highlight kotlin %}
class User {
    val allPayments : List<Payment> by lazy { fetchPaymentsFromDatabase() }
}

fun fetchPaymentsFromDatabase() : List<Payment> {
    //costly operation
    println("fetching payments from database")
    return listOf(Payment(1), Payment(2), Payment(3))
}
{% endhighlight %}

Yukarıdaki örnekte kullanılan `lazy delegation` thread-safe'tir. Eğer çalıştığınız ortamda birden fazla thread yoksa senkronizasyon maliyetleri ile uğraşmamak için alağıdaki gibi `LazyThreadSafetyMode.NONE` kullanabilirsiniz.

{% highlight kotlin %}
val allPayments : List<Payment> by lazy (LazyThreadSafetyMode.NONE) { fetchPaymentsFromDatabase() }
{% endhighlight %}


Bir Kotlin yazısının daha sonuna geldik. Detaylara indikçe bu dili daha çok sevmeye başlıyorum :)