---
layout: post
title: JUnit ve Docker Yardımıyla Kolay Entegrasyon Testleri
categories: development test junit docker
---
Yeni bir yazı serisine başlıyorum. Bu seride entegrasyon testlerini `JUnit` ve `Docker` yardımıyla nasıl yazdığımı anlatacağım.

Serinin ilk bölümü olan bu yazıda JUnit ve JUnit'in az bilinen fakat entegrasyon testleri yazarken kullanılması gereken bir özelliğinden, [@Rule](https://github.com/junit-team/junit4/wiki/Rules)'lardan bahsedeceğim. [JUnit5](https://junit.org/junit5/)'e henüz geçmediğim için örnekler [JUnit4](https://junit.org/junit4/) üzerinden olacak. Daha önce de [yazdığım](https://www.google.com.tr/search?q=kotlin+site%3Ailkinulas.github.io&oq=kotlin+site%3Ailkinulas.github.io) sebeplerden dolayı, bu yazı serisinde programlama dili olarak Kotlin kullanacağım.

Unit testlerin ortak ihtiyaçlarını kod tekrarı yapmadan ([DRY](http://wiki.c2.com/?DontRepeatYourself)) nasıl karşılarız? Bu soru 10 sene önce bana sorulmuş olsaydı cevabım  _inheritence_ olurdu. Bir tane `AbstracTest` sınıfı yaratırdım. Testlerin ortak ihtiyaçlarını bu abstract sınıfta tanımlardım ve tüm unit testlerimi bu `AbstractTest` sınıfından türetirdim. Fakat Object Oriented programlama dillerinin güçlü bir özelliği olan _inheritence_'ı _code reuse_ için kullanmamalıyız. _Code reuse_ söz konusu olduğunda öncelikle `composition`'ı düşünmemiz gerekir. 

> Bakınız "Effective Java 3rd Edition Item 18: Favor Composition Over Inheritence"

JUnit @Rule'ları sayesinde kod tekrarı yapmadan, testlerimize ortak kullanılan özellikleri ekleyebiliriz. Rule'ların çalışmasını kontrol eden iki farklı annotation var:
 * `@Rule` annotation'ı ile tanımlanan Rule'lar her test metodu için ayrı ayrı çalıştırılır.
 * `@ClassRule` annotation'ı ile tanımlanan Rule'lar her test class'ı için bir kere çalıştırılır.

Kendi Rule'umuzu yazmadan önce JUnit4 ile beraber gelen bazı Rule'lara göz atalım.

### TemporaryFolder Rule
Unit testlerin birbirinden `bağımsız` ve `izole` bir şekilde çalışmaları gerekiyor. Testlerden birisinin FileSystem'de yaptığı değişikliği diğer testlerin görmemesi lazım. Bunun için `TemporaryFolder` JUnit Rule'unu kullanabiliriz. Bu rule'da testler sırasında dosya oluşturmaya ve dizin oluşturmaya yardımcı metodlar bulunuyor. TemporaryFolder sınıfının public metodları ile oluşturduğumuz dosya ve dizinler testler tamamlandığında otomatik olarak silinirler.

Aşağıdaki `FileMerger.mergeFilesInFolder` metodu, istediğimiz bir dizin içindeki `prefix` ile başlayan dosyaları bulup, bu dosyaların içeriklerini `mergedFileName` adında ve aynı dizin içinde bulunan yeni bir dosyada birleştirir.

{% highlight kotlin %}
class FileMerger {
    fun mergeFilesInFolder(folder: String, prefix: String, mergedFileName: String) {   
        ....  
    }
}
{% endhighlight %}

Bu metodun istediğimiz gibi çalıştığından emin olmak için unit testlerini aşağıdaki gibi yazabiliriz.

{% highlight kotlin %}
//package & import declarations are omitted
class FileMergerTest {
    @Rule @JvmField val tempFolder = TemporaryFolder()

    @Test
    fun test_merge_files_success() {
        createTmpFile("a1.txt", 1..3)
        createTmpFile("a2.txt", 4..7)
        createTmpFile("a3.txt", 8..10)
        val expected = rangeToFileContent(1..10)
        val destination = "mergedfiles.txt"
        FileMerger().mergeFilesInFolder(tempFolder.root.path, "a*.txt", destination)
        val mergedFilePath = "${tempFolder.root.path}/$destination"
        assertEquals(expected, File(mergedFilePath).readText())
    }

    @Test
    fun test_merge_files_with_non_matching_prefix() {
        createTmpFile("a1.txt", 1..3)
        createTmpFile("a2.txt", 4..7)
        val destination = "mergedfiles.txt"
        FileMerger().mergeFilesInFolder(tempFolder.root.path, "b*.txt", destination)
        val mergedFilePath = "${tempFolder.root.path}/$destination"
        assertEquals("", File(mergedFilePath).readText())
    }

    private fun createTmpFile(fileName: String, range: IntRange) {
        tempFolder.newFile(fileName).bufferedWriter().use {
            it.write(rangeToFileContent(range))
        }
    }

    private fun rangeToFileContent(range: IntRange) = range.joinToString(separator = "\n", postfix = "\n")
}
{% endhighlight %}

Method @Rule'larının `public` @ClassRule'larının da `public static` olması gerekmektedir. `tempFolder` adındaki değişkenin başındaki `@JvmField` annotation'ı Kotlin compiler'ın bu değişken için getter/setter üretmemesini sağlar. Bu sayede JUnit framework'u bu değişkene erişebilir.

### System Rules
Junit @Rule'larına güzel bir örnek de [`System Rules`](http://stefanbirkner.github.io/system-rules/) paketi. Kendi Rule'larınızı yazmadan önce bu paket içindeki Rule'lara bakmanızı öneririm. Örneğin ortam (environment) değişkenlerini kullanan kodlarınız varsa, testlerinizde [`EnvironmentVariables`](https://stefanbirkner.github.io/system-rules/apidocs/org/junit/contrib/java/lang/system/EnvironmentVariables.html) Rule'unu kullanabilirsiniz. Testler sırasında bu Rule ile değiştirdiğiniz ortam değişkenleri test sonunda tekrar eski haline güncellenir.

{% highlight kotlin %}
@Rule @JvmField val env = EnvironmentVariables()

@Test
fun test_environment_variable_usage() {
    env.set("DB_TYPE", "mysql")
    //DB_TYPE adındaki ortam degiskeni sadece bu test icin "mysql" degerini alır.
    //Bu test bitince temizlenir diğer testler bu değişiklikten etkilenmez.
}
{% endhighlight %}

### Kendi Rule'umuzu Yazalım
Mevcut Rule'lar işimizi görmediği zamanlarda kendi Rule'larımızı yazabiliriz. Örneğin her @Test metodunun çalışmasının kaç milisaniye sürdüğünü loglamak için aşağıdaki gibi bir Rule yazabiliriz.

{% highlight kotlin %}
class TestDurationRule : TestRule {
    override fun apply(base: Statement, description: Description): Statement {
        return object : Statement() {
            override fun evaluate() {
                val start = System.currentTimeMillis()
                try {
                    base.evaluate()
                } finally {
                    val duration = System.currentTimeMillis() - start
                    println("${description.methodName} took $duration ms")
                }
            }
        }
    }
}
{% endhighlight %}

`TestDurationRule`'u aşağıdaki gibi testlerimize ekleyebiliriz. @Rule olarak kullanırsak her metod için ayrı ayrı ölçüm yapılır. @ClassRule olarak kullandığımızda ise test sınıfı içindeki tüm metodların toplam çalışma süresi loglanır.

{% highlight kotlin %}
@Rule @JvmField val methodDuration = TestDurationRule()

companion object {
    @ClassRule @JvmField val classDuration = TestDurationRule()
}
{% endhighlight %}

JUnit test Rule'ları ile başladığımız bu yazı dizisini Docker'a nasıl bağlayacağımı merak ediyorsanız ufak bir ipucu vereyim : [Testcontainers](https://www.testcontainers.org/). Rule'lar sayesinde testler başlamadan önce Docker container'larını ayağa kaldırıp testlerimizde kullanacağız, testler tamamlandığında da container'ları kapatacağız. Bir sonraki yazıda buluşmak üzere. 

> Kodunuzu automated test'lerden mahrum etmeyin! -anonim