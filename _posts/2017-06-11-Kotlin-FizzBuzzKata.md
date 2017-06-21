---
layout: post
title: Kotlin ile FizzBuzz
categories: development kotlin
---
[Google IO 2017](https://events.google.com/io/)'de uygulama geliştiricileri en çok heyecanlandıran haber Google'ın resmi olarak Kotlin programlama diline destek vermesi [oldu](https://www.youtube.com/watch?v=X1RVYt2QKQE). Bugüne kadar hakettiği ilgiyi göremeyen **Kotlin**, Google IO 2017 etkinliğinden sonra özellikle android developer'lar arasında hızla [popüler olmaya başladı](https://trends.google.com/trends/explore?cat=5&date=all&q=kotlin). 

Kotlin öğrenmesi kolay bir programla dili. Bir java developer'ın [Kotlin dokümanlarını](https://kotlinlang.org/docs/reference/) okuyup Kotlin biliyorum demesi için bir haftasonunu bu işe ayırması bence yeterli olur. Fakat diğer programlama dillerinde de olduğu gibi [ustalaşmak](http://manifesto.softwarecraftsmanship.org/) için sadece dokümanları ya da blog post'ları okumak yeterli olmuyor. 

Sanatçılar ve sporcular için pratik yapmak ne kadar önemliyse biz yazılım geliştiriciler için de pratik yapmak o kadar önemlidir. "Bütün gün iş yerinde kod yazarak pratik yapıyorum zaten" diye düşünmeyin. Bir yazılımcının mesai saatleri içerisinde yaptığı işler onun pratiği değil ortaya koyduğu performanstır. NBA'de oynayan bir basketbolcu için resmi maçlar onun performans yeridir. Bir ses sanatçısı için binlerce insan önünde verdiği konser bir performans yeridir. Çalıştığımız şirketler de biz yazılımcılar için bir performans yeridir. 

Pratik yaparken istediğimiz kadar hata yapmakta serbestiz. Zaten pratik yapmanın da amacı _hata yapa yapa hatasız bir performans ortaya koymak_ değil midir? Ben dahil çoğu yazılım geliştirici arkadaşın içine düştüğü bir tuzak pratiklerimizi çalıştığımız şirketlerdeki projelerde yapmamız. Bu yüzden işlerimizi yaparken bu kadar çok "amatör" hatalar yapıyoruz. 

Peki bir yazılım geliştirici nasıl pratik yapabilir? Çok basit, tabi ki kod yazarak. Kendimize küçük problemler bulup her gün yarım saat ya da bir saat bu problemleri favori programlama dilimizde yazacağımız kod parçaları ile çözmeye çalışabiliriz. Hergün çözülecek yeni bir problem bulmaya da gerek yok. Aynı problemi defalarca her defasında farklı yaklaşımlarla çözmeye çalışabiliriz. Kullandığımız IDE'nin kısayollarını öğrenip mouse kullanmadan kod yazmaya çalışmak bile yapılabilecek iyi bir pratiktir. (Mouse kullanmadan kod yazmak için kendinizi zorlayın, harcadığınız emeğe değeceğini göreceksiniz.) 

Karete ustaları ustalaşmak için çeşitli vuruş ve bloklardan oluşan bir dizi hareketi, bu hareketlerde mükemmelleşene kadar tekrar ederler. Karate ustalarının uyguladıkları bu yöntemi yazılım ustaları olarak bizler de uygulayabiliriz. Yazılım geliştiricilerin sanatlarında ustalaşmak için yaptığı bu çalışmalara **Kod Kata** [deniliyor](http://codekata.com/). Kod kata ile ilgili [kurumsaljava.com](http://www.kurumsaljava.com/2012/04/07/kod-kata-ve-pratik-yapmanin-onemi) sitesinde güzel bir türkçe yazı var. Bu yazıdan sonra onu da okumanızı tavsiye ederim.

![Karate Kata](/assets/kotlin_fizzbuzz/kata.png)

Bu yazıda, en populer kod kata problemlerinden birisi olan __[FizzBuzz](https://en.wikipedia.org/wiki/Fizz_buzz)__'i Kotlin programlama dilini kullanarak çözmeye çalışacağız. FizzBuzz bir sayı sayma oyunu. Saymaya 1 rakamından başlanır ve sırası gelen sayı 3'e kalansız bölünebiliyorsa __Fizz__ 5'e kalansız bölünebiliyorsa __Buzz__, hem 3'e hem de 5'e bölünebiliyorsa __FizBuzz__ denir. Aşağıda örnek bir FizzBuzz sayı dizisi görüyorsunuz.

> 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, FizzBuzz, 16 ...

Amacımız N'e kadar olan sayılar için FizzBuzz dizisi üreten bir fonksiyon yazmak. Fonksiyonun adı, aldığı parametreler ve dönüş tipi aşağıdaki gibi olsun. Bu fonksiyonu Kotlin programlama dilini kullanarak kaç farklı şekilde yazabileceğimize bir bakalım.

{% highlight kotlin %}
fun fizzBuzz(n : Int) : String {...}
{% endhighlight %}

## 1. İlk akla gelen yöntem

İlk çözüm için çok fazla düşünmeden basit bir __for loop__ yazalım. Aşağıdaki kod örneğini anlamak için Kotlin bilmeye gerek yok.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val sb = StringBuilder()
    for (i in 1..n) {
        if (i % 15 == 0) {
            sb.append("FizzBuzz")
        } else if (i % 3 == 0) {
            sb.append("Fizz")
        } else if (i % 5 == 0) {
            sb.append("Buzz")
        } else {
            sb.append(i)
        }
    }
    return sb.toString()
}
{% endhighlight %}

## 2. __when__ Expression 
İkinci çözümde __for loop__'a ilave __when expression__ da kullanıyoruz. Artık yavas yavas Kotlin'in güzelliklerini görmeye başladık. __if / else if__ bloklarını daha okunaklı __when__ expression ile değiştirirsek aşağıdaki gibi bir sonuç elde ederiz.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val sb = StringBuilder()
    for (i in 1..n) {
        when {
            i % 15 == 0 -> sb.append("FizzBuzz")
            i % 3 == 0 -> sb.append("Fizz")
            i % 5 == 0 -> sb.append("Buzz")
            else -> sb.append(i)
        }
    }
    return sb.toString()
}
{% endhighlight %}

## 3. Standard library'den __apply__ fonksiyonu
Kotlin standard library'den bahsettiğim yazıya [buradan](/development/kotlin/2017/03/04/Kotlin-Standard-Library.html) ulaşabilirsiniz.
__apply__ extension metodu, uygulandığı instance’ı receiver olarak alan bir lambda expression çalıştırır ve sonuç olarak aynı instance’ı geri döner. _StringBuilder_'a uygulandığında lambda içinde StringBuilder referansına ihtiyaç duymadan "append" metodunu çağırabiliriz.

Aşağıdaki örnek'te dikkatinizi çekmek istediğim bir şey daha var. Kotlin'de metod'lar tek bir "expression"'dan oluşuyorsa metodun dönüş değerini, süslü parantezleri ve "return" keyword'ünü yazmaya gerek yok. "=" işaretinden sonra metod gövdesini yazabiliriz.

{% highlight kotlin %}
fun fizzBuzz(n: Int) = StringBuilder().apply {
    for (i in 1..n) {
        when {
            i % 15 == 0 -> append("FizzBuzz")
            i % 3 == 0 -> append("Fizz")
            i % 5 == 0 -> append("Buzz")
            else -> append(i)
        }
    }
}
{% endhighlight %}

## 4. Sequence, map ve zip
Bundan önceki örneklerde problemin çözümü için prosedürel yöntemler kullandık. Kotlin, JVM üzerinde fonksiyonel programlama yapmaya olanak veren bir programlama dili. İsterseniz yavas yavas Kotlin'in bu özelliğini kullanmaya başlayalım. Kotlin standard library'de [Collection](/development/kotlin/2017/03/10/Kotlin-Collections.html)'lar ve [Sequence](/development/kotlin/2017/03/26/Kotlin-Sequences.html)'lar üzerinde kullanabileceğimiz __map__ ve __zip__ gibi higher-order fonksiyonlar mevcut.

Aşağıdaki örnekte önce 4 farklı sequence oluşturuyoruz. Bu sequence'ları __zip__ fonksiyonu ile tek bir sequence'a indirgeyip, sequence'in ilk "n" elemanından __joinToString__ metodu ile sonucu oluşturuyoruz. Sizi bilmem ama __for loop__ ve __if/else__ kullanmadığımız bu çözüm benim hoşuma gitti.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val numbers = generateSequence(1) { it + 1 }.map { it.toString() }
    val fizz = generateSequence(1) { it + 1 }.map { if (it % 3 == 0) "Fizz" else "" }
    val buzz = generateSequence(1) { it + 1 }.map { if (it % 5 == 0) "Buzz" else "" }
    val fizzBuzz = generateSequence(1) { it + 1 }.map { if (it % 15 == 0) "FizzBuzz" else "" }
    val max = { a: String, b: String -> if (a > b) a else b }
    return numbers
            .zip(fizz, max)
            .zip(buzz, max)
            .zip(fizzBuzz, max)
            .take(n)
            .joinToString(", ")
}
{% endhighlight %}

## 5. Tekrar eden sequence'lar
Bu çözüm bir önceki çözüme çok benziyor. Standard library'de tekrar eden bir sequence oluşturmaya yarayan bir extension method olmadığı için kendi __RepeatingSequence__'ımızı yazacağız. Aşağıda her 3 elamanınından birisi "Fizz" ve her 5 elamanından birisi "Buzz" olarak terkar eden diziler var.

```
Fizz Sequence : "", "", Fizz, "", "", Fizz, "", "", Fizz, ...
Buzz Sequence : "", "", "", "", "Buzz", "", "", "", "", "Buzz", "", "", "", "", "Buzz", ...
```

Mevcut bir sequence'ı alıp onu tekrar eden bir sequence haline getirmek için _Sequence_ sınıfından türettiğimiz aşağıdaki_RepeatingSequence_ sınıfını kullanabiliriz. Herhangi bir sequence'ı alıp onu tekrar edecek hale getirebilmek için de aşağıdaki _repeat()_ adındaki extension fonksiyonunu kullanıyoruz.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val numbers = generateSequence(1) { it + 1 }.map { it.toString() }
    val fizz = listOf("", "", "Fizz").asSequence().repeat()
    val buzz = listOf("", "", "", "", "Buzz").asSequence().repeat()
    val fizzBuzz = fizz.zip(buzz) { a, b -> a + b }
    val max = { a: String, b: String -> if (a > b) a else b }
    return numbers.zip(fizzBuzz, max).take(n).joinToString(", ")
}

class RepeatingSequence<T>(val sequence: Sequence<T>) : Sequence<T> {
    var iterator = sequence.iterator()
    override fun iterator(): Iterator<T> = object : Iterator<T> {
        override fun hasNext(): Boolean = iterator.hasNext()

        override fun next(): T {
            val result = iterator.next()
            if (!iterator.hasNext()) {
                iterator = sequence.iterator()
            }
            return result
        }
    }
}

fun <T> Sequence<T>.repeat() = RepeatingSequence(this)
{% endhighlight %}

## 6. Sequence, lambda'lar ve String Birleştirme
Aşağıdaki çözüm kendi kendini anlatıyor. Her rakamı  __fizz__, __buzz__ ve __number__ lambda'ları ile String'e dönüştürüyoruz ve oluşan String'leri birleştiriyoruz.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val fizz = { i: Int -> if (i % 3 == 0) "Fizz" else "" }
    val buzz = { i: Int -> if (i % 5 == 0) "Buzz" else "" }
    val number = { i: Int -> if (i % 3 != 0 && i % 5 != 0) i else "" }
    return generateSequence(1) { it + 1 }.map {fizz(it) + buzz(it) + number(it)}.take(n).joinToString(", ")
}
{% endhighlight %}

## 7. Hardcoded Array Index'leri

Bu çözüm diğer çözümlere göre biraz __ilginç__. 15 elemanlı __fizzBuzz__ adındaki integer dizisinin elemanlarını __map__ fonksiyonu içindeki geçici dizinin ```arrayOf(it, "Fizz", "Buzz", "FizzBuzz")``` index'i olarak kullaniyoruz. 

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    val fizzBuzz = intArrayOf(3, 0, 0, 1, 0, 2, 1, 0, 0, 1, 2, 0, 1, 0, 0)
    return generateSequence(1) { it + 1}
            .map {  arrayOf(it, "Fizz", "Buzz", "FizzBuzz")[fizzBuzz[it % 15]] }
            .take(n).joinToString(", ")
}
{% endhighlight %}

## 8. Map veriyapısı yardımıyla çözüm

Aşağıdaki çözümde her iterasyonda yeni bir map instance'ı yaratılıyor. Bu çözümü iyi bir çözüm olduğu için eklemedim. Düşündüğümüzde ne kadar farklı şekillerde çözümler üretilebileceğini göstermek amacım.

{% highlight kotlin %}
fun fizzBuzz(n: Int): String = (1..n).map {
        mapOf(0 to it,
            it % 3 to "Fizz",
            it % 5 to "Buzz",
            it % 15 to "FizzBuzz")[0] }.joinToString(", ")

{% endhighlight %}

Aynı fonksiyonu aşağıdaki gibi tek satır olarak yazabiliriz. Beraber çalıştığınız insanları seviyorsanız bunu yapmayın :)

{% highlight kotlin %}
fun fizzBuzz(n: Int): String = (1..n).map { mapOf(0 to it, it % 3 to "Fizz", it % 5 to "Buzz", it % 15 to "FizzBuzz")[0] }.joinToString(", ")
{% endhighlight %}

## 9. Magic Number (2548493554) çözümü

Yazıyı buraya kadar sıkılmadan takip edenler için bonusu en sona sakladım :)

{% highlight kotlin %}
fun fizzBuzz(n: Int): String {
    var random = Random()
    return generateSequence(1) { it + 1 }.map {
        if (it % 15 == 1) random.setSeed(2548493554)
        listOf(it.toString(), "Fizz", "Buzz", "FizzBuzz")[random.nextInt(4)]
    }.take(n).joinToString(", ")
}
{% endhighlight %}

Peki bu 2548493554 sayısı nereden çıktı? 

```random.nextInt(4)``` metodunu peş peşe 15 kez çağırdığımızda bize sırasıyla ```0, 0, 1, 0, 2, 1, 0, 0, 1, 2, 0, 1, 0, 0, 3``` değerlerini vermesi için hangi __seed__'i kullanmamız gerektiğini bulursak sihirli sayıya ulaşmış oluruz. Her 15 iterasyonda bir random number generator'u resetlersek (setSeed) bize sürekli aynı diziyi üretir. Bu sayıyı bulmak için de aşağıdaki gibi brute force bir yöntem kullanabiliriz. 

Bu örneği çalıştırdığınız runtime'ın random number generator'üne göre bulacağınız magic number buradakinden farklı olabilir.

{% highlight kotlin %}
fun magic(rnd: Random): Boolean {
    val fizzBuzz = intArrayOf(0, 0, 1, 0, 2, 1, 0, 0, 1, 2, 0, 1, 0, 0, 3)
    fizzBuzz.forEach {
        if (it != rnd.nextInt(4)) return false
    }
    return true
}

fun bruteForceMagicRandom() {
    val magicSeed = generateSequence(0L) { it + 1L }.filter {
        val rnd = Random(it)
        magic(rnd)
    }.take(1).toList()
    println(magicSeed)
}

fun main(args: Array<String>) {
    bruteForceMagicRandom()
}
{% endhighlight %}

İşte gördünüz, FizzBuzz gibi basit bir problem için bile 9 farklı çözüm ortaya çıktı ve bu çözümlerin her birinde Kotlin programlama dilinin farklı özelliklerini kullanma fırsatı bulduk. Siz de benim gibi mesleğini seven bir yazılımcıysanız ve iyi bir performans ortaya koymak istiyorsanız bol bol pratik yapın ve kod kata yapmayı alışkanlık haline getirin.