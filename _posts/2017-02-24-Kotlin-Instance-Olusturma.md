---
layout: post
title: Kotlin ile "Instance" Oluşturma
categories: development kotlin
---
Bu yazıda `Constructor`'lardan ve class instance'ları oluşturmanın farklı yöntemlerinden bahsedeceğim.

Kotlin'de sınıflar *bir* tane `primary constructor`'a ve bir veya birden fazla `secondary constructor`'a  sahip olabilirler. Aşağıda 2 parametre alan (name ve speed) primary constructor örneği görüyorsunuz. Primary constructor tanımı class isminin hemen ardından başlar. Constructor parametrelerinin başındaki `val` ve `var` sayesinde Kotlin compiler _name_ ve _speed_ adında property'leri otomatik olarak oluşturur.

{% highlight kotlin %}
class Monster(val name: String, var speed: Int)

fun main(args: Array<String>) {
    val monster = Monster("werewolf", 10)
    println("Monster speed is ${monster.speed}") // prints 'Monster speed is 10'
}
{% endhighlight %}

Primary constructor'in tüm parametreleri için default değerler tanımlanırsa kotlin compiler parametre almayan ve field'lara default değerleri atayan bir constructor daha yaratır. 

{% highlight kotlin %}
class Weapon(val damage: Int = 10, val range: Int = 100)

fun main(args: Array<String>) {
    val crossBow = Weapon() // damage = 10 & range = 100
    val longRangeCrossBow = Weapon(20, 200) // damage = 20 & range = 200
}
{% endhighlight %}

Primary construct içine kod yazılmaz. Init işlemleri için  aşağıdaki örnekte olduğu gibi `initializer block`'lar kullanılır.

{% highlight kotlin %}
class Monster(val name: String, var speed: Int) {
    init {
        //Initialization code ...
    }
}
{% endhighlight %}

Secondary constructor'lar birden fazla olabilir ve tanımlanırken aşağıdaki örnekteki gibi `constructor` keyword kullanılır. Class'ın bir primary constructor'ı varsa primary constructor'a `this` ile gönderme yapmak gerekir:

{% highlight kotlin %}
class Node(name: String) {
    constructor(name: String, parent: Node) : this(name)
}
{% endhighlight %}

## Static Factory Method

Aksini belirtmediğimiz sürece Kotlin'de constructor'lar `public`'tir. Aşağıdaki örnekte constructor'ı gizlenmiş bir class örneği görebilirsiniz: 

{% highlight kotlin %}
class Spell private constructor(val level:Int, val duration:Int) {
    companion object {
        fun acidSplash() = Spell(1, 10)
        fun invisibility() = Spell(2, 20)
    }
}

fun main(args: Array<String>) {
    //val spell = Spell(1, 10) // compile error
    val acidSpell = Spell.acidSplash()
    val invisibilitySpell = Spell.invisibility()
}
{% endhighlight %}

Bu örnekte, oluşturmak istedigimiz sınıfın instance'ını dönen public `static factory method`'lar görüyorsunuz. Constructor private olduğu için `Spell` oluşturmak istedigimiz zaman bu factory method'ları kullanmamız gerekir. Static factory method'ların `Effective Java Item 1`'de anlatılan avantajları şunlar:

* Static factory method'lara anlamlı isimler verebiliriz. `acidSplash` ve `invisibility` gibi.
* Constructor'lar her çağırıldıklarında yeni bir instance oluşur, fakat static factory method'lar her çağırıldıklarında yeni bir instance yaratmak zorunda değildir. Aşağıdaki örnekte olduğu gibi daha önceden yaratılmış ve cache'lenmiş bir instance dönebilirler. 

{% highlight kotlin %}
class Error private constructor(val code: Int, val message: String) {
    init {
        println("Error $code $message created.")
    }
    companion object {
        private val errors: MutableMap<Int, Error> by lazy {
            mutableMapOf<Int, Error>()
        }

        private fun getOrCreateError(code: Int, message: String) =
                errors.getOrPut(code, {Error(code, message)})

        fun _401() = getOrCreateError(401, "Unauthorized")
        fun _404() = getOrCreateError(404, "Not found")
    }
}
{% endhighlight %}

> Joshua Bloch tarafından yazılan Effective Java kitabını mutlaka bulun okuyun, çok ciddiyim. Hatta henüz bu kitabı okumadıysanız bırakın bu yazıyı okumayı kitabın bir kopyasını edinin ve okumaya başlayın.

Yukarıdaki örnek'te Kotlin'e özel iki özellik var:

### 1. Lazy : `Delegated properties`
{% highlight kotlin %}
private val errors: MutableMap<Int, Error> by lazy {mutableMapOf<Int, Error>() }
{% endhighlight %}
_Error_ sınıfının _errors_ field'ının oluşması `lazy` tanımlandı. Bu şu demek oluyor, _errors_ adındakı `map`'e ilk kez erişildiğinde yeni bir _map_ instance yaratılır ve daha sonraki erişimlerde bu instance kullanılır. Bu mekanizmaya Kotlin'de [Delegated Properties](https://kotlinlang.org/docs/reference/delegated-properties.html) adı veriliyor. Delegate'lerden başka bir yazıda daha detaylı bahsedeceğim.

### 2. Zengin Standart Library
[`mutableMapOf`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-map-of.html) ve [`Colletions.getOrPut`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-put.html) Kotlin'in zengin standart library'si içindeki metodlardan sadece ikisi. Standart library fonksiyonları ile ilgili ayrı bir yazı yazmak istiyorum. 

## Builder Pattern

Karmaşık sınıfların instance'larını oluşturmak istediğimizde bir çok opsiyonel parametresi olan constructor'lar ya da static factory method'lar iyi bir çözüm olmaz. Böyle durumlarda `Builder` tasarım kalıbını kullanabiliriz. 

Yine bir örnek üzerinden gidelim. Kullanıcılara uyarı mesajı göstermek için kullanacağımız bir `Dialog` sınıfımız olsun. Başlık, mesaj, renkler ve dialog kapatıldığında çalıştırmak istediğimiz bir code bloğunu parametre olarak alan bir constructor aşağıdaki gibi tanımlanabilir.

{% highlight kotlin %}
class Dialog(
        val title: String,
        val message: String,
        val titleColor: Color,
        val bodyColor: Color,
        val onClose: () -> Unit) {
}
fun main(args: Array<String>) {
    val dialog = Dialog(
            "Baglanti Problemi",
            "Internet baglantinizi kontrol edip tekrar deneyin.",
            Color.RED,
            Color.BLACK,
            { println("Uyari mesaji kapatildi.") })
}

{% endhighlight %}

Sadece `title` ve `message` parametreleri verip default renklerde ve kapatıldığında ekstra birşey yapmasını istemediğimiz bir dialog oluşturmak istersek aşağıdaki gibi bir secondary constructor ekleyebiliriz.

{% highlight kotlin %}
class Dialog(
        val title: String,
        val message: String,
        val titleColor: Color,
        val bodyColor: Color,
        val onClose: () -> Unit) {

    constructor(title: String, message:String) :
            this(title, message, Color.RED, Color.YELLOW, {})
}
fun main(args: Array<String>) {
    val d2 = Dialog("Tebrikler", "Oyunu kazandiniz.")
}
{% endhighlight %}

Şimdi dialog oluşturmak biraz daha kolay. Fakat bu sefer de farklı renklerde dialog'lar oluşturmak istersek yeni constructor'lar eklemek zorunda kalacağız ya da 5 tane parametre alan primary constructor'ı kullanmaya devam edeceğiz. Dialog sınıfına zamanla yeni özellikler eklediğimizde (ikon, açılma animasyonu, kapanma animasyonu gibi...) client'ların constructor ile dialog oluşturması karmaşıklaşmaya başlar. Tam burada `Builder` pattern yardımımıza koşuyor.

Kotlin'de builder yazmanın farklı yöntemleri var. Aşağıdaki örnekte `extension` metodlarını kullanan `DialogBuilder` sınıfı var. Bu builder sayesinde:

* Client'lar icin dialog oluşturmak kolaylaştı ve client kodu daha okunaklı oldu.
* Dialog sınıfına opsiyonel property'ler eklemek daha kolay oldu.

<script src="https://gist.github.com/ilkinulas/00a44e120514c1235aa7c0f36b6659fb.js"></script>

Yukarıdaki örnekte 31. satırdaki build metodunun 3. parametresini beraber inceleyelim: 

`init: DialogBuilder.() -> Unit` . `init` adındaki bu function'ın tipi `DialogBuilder.() -> Unit`'dir. Kotlin jargon'unda bu tür fonksiyonlara `function type with receiver` deniliyor. [Linkteki](https://kotlinlang.org/docs/reference/type-safe-builders.html) dokümanda detayları anlatılan  bu tip fonsiyonlar `Type Safe Builder`'ları yazmaya olanak sağlar. 

Tekrar bizim örneğimize dönecek olursak; `init` fonksiyonu parametre olarak DialogBuilder instance'i (receiver) istiyor ve bu instance'in public metodlarına ve property'lerine  `init` fonksiyonu içinden erişebiliyoruz. 40, 41 ve 42. satırlarda yapılan işlemlerin `receiver`'i DialogBuilder olduğu için `titleColor = Color.RED` dediğimiz zaman DialogBuilder'ın `title` field'ını güncellemiş oluyoruz. 

Son olarak, aşağıdaki kullanım örneklerine bakarak, builder pattern'in  karmaşık instance'ları oluştururken okuması ve kullanımı kolay kod yazmamıza yardım ettiğini açıkça görebiliriz.

{% highlight kotlin %}
val errorDialog = Dialog.build("Uzgunuz", "Beklenmedik bir hata olustu") {
    titleColor = Color.RED
    bodyColor = Color.YELLOW
    icon = Image.error()
}

val infoDialog = Dialog.build("Tebrikler", "Buyuk ikramiye size cikti.") {
    titleColor = Color.GREEN
    icon = Image.info()
}

val greetingDialog = Dialog.build("Selam", "Hosgeldiniz")
{% endhighlight %}