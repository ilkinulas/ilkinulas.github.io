---
layout: post
title: Kotlin Collections
categories: development kotlin
excerpt_separator: <!--more-->
---
Kotlin'de collection'lar, java collection library'leri üzerine yazılmış yardımcı sınıflar ve extension methodlardan oluşur. Kotlin programlama dilinin tasarımcıları, `list`, `map` ve `set` gibi veriyapılarını sıfırdan yazmak yerine, Kotlin collection altyapısını, mevcut java sınıflarının üzerine kurma kararı vermişler. 

Kotlin projesinin başındaki adam Andrey Breslav'dan bir alıntı yaparak kod örneklerine geçelim: 

>  “Kotlin relies on Java libraries but makes them better, mostly through extensions but
>  sometimes with compiler-supported techniques (collections, arrays, primitives).” 
>  - Andrey Breslav, JetBrains

<!--more-->

Kotlin'de collection'lar siz aksini belirtmedikçe `read-only`'dir. Liste oluşturmak için standard library içindeki `listOf` ve `mutableListOf` metodlarını kullanabiliriz.

`listOf` metodu [`List<out T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/) tipinde bir liste döner. `listOf` metodundan aldığımız referans ile listeye eleman ekleyip çıkartamayız.

{% highlight kotlin %}
val numbers = listOf(0, 1, 2, 3)
val fruits = listOf("apple", "orange", "banana")

println(numbers[2]) // prints 2, same as 'numbers.get(2)'
println(fruits.size) // prints 3
{% endhighlight %}

İçeriğini zaman içinde değiştirmek istediğimiz bir liste oluşturmak istersek `mutableListOf` metodunu kullanabiliriz, bu metod bize [`MutableList`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list.html) tipinde bir referans döner.

{% highlight kotlin %}
val achievements = mutableListOf("Rookie", "Talented")
achievements.add("Pro")
achievements.removeAt(0)
achievements.addAll(listOf("Expert", "Star"))
{% endhighlight %}

Kotlin'de şimdilik `immutable` collection'lar yok. Ben bu yazıyı hazırlarken en son Kotlin versiyonu 1.1'di. `Read-only` ve `immutable` kavramlarını karıştırmamak lazım. `immutable` bir collection bir kere oluşturulduktan sonra güncellenemez. Örnek olarak [Guava Immutable Collections](https://github.com/google/guava/wiki/ImmutableCollectionsExplained) sınıflarına bakabilirsiniz.

Bizim elimizde `mutable` bir collection'ı gösteren read-only bir referans olabilir. Biz read-only referansımız ile collection'ı güncelleyemeyiz fakat alt taraftaki `mutable` collection bizim kontrolümüz dışında güncellenebilir. Aşağıdaki örnekte `List<out T>`'nin immutable değil read-only bir tip olduğunu görebilirsiniz:

{% highlight kotlin %}
val movies = mutableListOf("Batman", "Fight Club", "Matrix")
val readOnlyMovies: List<String> = movies
//readOnlyMovies.add("Forrest Gump") --> compile error!

println(readOnlyMovies) // prints [Batman, Fight Club, Matrix]

movies.clear()
println(readOnlyMovies) // prints []

//Dikkat !! read-only ozelligi runtime'da ise yaramayabilir.
(readOnlyMovies as MutableList<String>).add("The Godfather")
println(readOnlyMovies) // prints [The Godfather]
{% endhighlight %}

Aşağıdaki örnek de Kotlin-Java geçişi sırasınra listenin read-only özelliğini bir işe yaramayacağını gösteriyor.

{% highlight kotlin %}
//Kotlin
data class Movie(val name: String)

class MovieDb {
    val movies = listOf(Movie("Batman"), Movie("Fight Club"), Movie("Matrix"))
}

fun main(args: Array<String>) {
    val movieDb = MovieDb()
    //movieDb.movies.add(Movie("Alien")) // --> compile error!

    CollectionsJava.mutateDb(movieDb, Movie("Alien"))
    //movie listesinin ilk elemanini "Alien" ile degistirdi.
}


//Java
public class CollectionsJava {
    public static void mutateDb(MovieDb db, Movie movie) {
        db.getMovies().set(0, movie);
    }
}
{% endhighlight %}

Kotlin collection'larının Java'dakilerden bir farkı da `covariant` olmalarıdır. Java'da generic collection'lar `invariant`'tır. Örneğin java'da `List<Number>` tipindeki bir referansa `List<Integer>` tipinde bir referans ataması yapamayız.

Aşağıdaki java örneğini derlemeye çalıştığımızda `Incompatiple Types` hatası alırız. Bu sayede java compiler `numbers`referansı üzerinden Integer listesine `Double` tipinde bir eleman eklenmesini önlemiş olur.

{% highlight kotlin %}
List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5);
List<Number> numbers = ints; // compile error!!
{% endhighlight %}

Java `array`'leri `covariant`'tır. Aşağıdaki kod problemsiz bir şekilde derlenir fakat çalıştırıldığında `java.lang.ArrayStoreException` fırlatır.

{% highlight kotlin %}
Integer[] intArray = new Integer[]{1, 2, 3, 4, 5};
Number [] numberArray = intArray;

numberArray[0] = new Double(Math.PI); // throws java.lang.ArrayStoreException
{% endhighlight %}

Kotlin collection'ları `covariant`olduğu için Integer listesini Number listesi yerine kullanabiliriz. Fakat Number listesi arayüzünden eriştiğimiz Integer listesini değitiremeyiz (ekleme ve çıkarma yapamayız).

{% highlight kotlin %}
var ints = listOf(1, 2, 3, 4)
var numbers : List<Number> = ints
{% endhighlight %}

Covariance ve Invariance hakkında daha detaylı bilgi için `Effective Java` kitabının 5. bölümüne (Generics) bakabilirsiniz.    

Kotlin standard library'de bilmemiz gereken ve collection'lar ile çalışmayı kolaylaştıran birçok extension metod var. `map`, `flatMap`, `fold`, `filter`, `distinct`, `average`, `groupBy`, `reduce` bu extension'lardan sadece bazıları.

Tek tek bu extension metodlarının ne yaptığını anlatmaya gerek yok, merak edenler Kotlin standard library kaynak kodlarına bakabilirler. Bu metodların developer'ların işini ne kadar kolaylaştırdığını göstermek için gerçek bir örnek vermek istiyorum.

[Flickr](http://www.flickr.com/) çok eski bir fotoğraf paylaşım sitesi ve servislerini bir REST API üzerinden açıyor. Popüler resimleri sorgulamak için çağırılan bir servis aşağıdaki gibi bir json data dönüyor. (Çok fazla yer kaplamaması için json'ın bir kısmını kestim.)

<pre>
{ "photos": { "page": 1, "pages": "3207", "perpage": 100, "total": "320662",
    "photo": [
      { "id": "32547078274", "owner": "148578535@N03", "secret": "9f61ddcc9f", "server": "3855", "farm": 4, "title": "", "ispublic": 1, "isfriend": 0, "isfamily": 0 },
      { "id": "33007015650", "owner": "148578535@N03", "secret": "9031056fa7", "server": "660", "farm": 1, "title": "", "ispublic": 1, "isfriend": 0, "isfamily": 0 },
      ....
      { "id": "32547072484", "owner": "124285120@N06", "secret": "8f8de22696", "server": "3893", "farm": 4, "title": "Photo", "ispublic": 1, "isfriend": 0, "isfamily": 0 }
    ] }, "stat": "ok" }
</pre>

Amacımız bu json'ı parse edip her foto için URL oluşturmak. Json işlemleri için ekstra bir kütüphane kullanmaya gerek yok. Android runtime içinde gelen `org.json` kütüphanesi işimizi görür. Kotlin ve Java'nın bir kardeş gibi anlaştıklarından (Interoperability) ve Kotlin kodu içinden java ile yazılmış kütüphaneleri pürüzsüz bir şekilde kullabildiğimizden daha önceki yazılarda [bahsetmiştim](/development/kotlin/2017/01/28/Neden-Kotlin.html)

[JSONArray](https://developer.android.com/reference/org/json/JSONArray.html) sınıfı çok primitive bir sınıf. Elemanlarını dolaşmak için bir `iterator` sunmuyor. İlk yapacağımız iş JSONArray için bir iterator extension metodu yazmak olacak:

{% highlight kotlin %}
operator fun JSONArray.iterator(): Iterator<JSONObject>
        = (0..length() - 1).asSequence().map { get(it) as JSONObject }.iterator()
{% endhighlight %}

Artık JSONArray elemanlarını `forEach` ile gezebiliriz:

{% highlight kotlin %}
photosJsonArray.iterator().forEach {  ...  }
{% endhighlight %}

Fakat `map` ve `filter` gibi extension'lar `Iterable<T>` tipi üzerine tanımlandığı için bize JSONArray'ı Iterable'a dönüştürecek bir extension lazım:

{% highlight kotlin %}
fun JSONArray.iterable(): Iterable<JSONObject> = Iterable { iterator() }
{% endhighlight %}

Son vuruşu `map` fonksiyonu ile yapıyoruz. `map`sayesinde JSONObject'leri url string'lere dönüştüreceğiz.

{% highlight kotlin %}
val json = JSONObject(jsonString)
val photosJsonArray = json.getJSONObject("photos").getJSONArray("photo")

val photoUrls = photosJsonArray.iterable().map {
    "https://farm${it["farm"]}.staticflickr.com/${it["server"]}/${it["id"]}_${it["secret"]}.jpg"
}
photoUrls.forEach {
    println(it)
}
{% endhighlight %}

Java'da olmayan bir başka Kotlin güzelliği de url string'i oluştururken kullandığımız [`string interpolation`](https://kotlinlang.org/docs/reference/basic-types.html#string-templates) yöntemi.

Ve sonuç olarak populer fotoların linklerine ulaşmış oluyoruz.

<pre>
https://farm4.staticflickr.com/3855/32547078274_dcc9f61d9f.jpg
https://farm1.staticflickr.com/660/33007015650_10a9056f37.jpg
https://farm4.staticflickr.com/3895/33262233881_5a583dd624.jpg
https://farm4.staticflickr.com/3948/32547078094_e751d10e40.jpg
...
</pre>

Bir sonraki yazida [Sequence](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/)'lardan yani `lazy` collection'lardan bahsedeceğim.