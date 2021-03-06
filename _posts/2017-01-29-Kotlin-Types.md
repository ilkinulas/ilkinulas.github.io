---
layout: post
title: Kotlin Types
categories: development kotlin
---
> Teknik konular hakkında Türkçe yazmak gerçekten çok zor. Bir çok kelimenin Türkçe karşılığını 
> bulmakta zorlanıyorum. Yarı Türkçe yarı İngilizce yazmak hiç hoşuma gitmese de aklımdakileri bir 
> an önce yazıya döküp paylaşmak için şimdilik çok fazla kafaya takmadan yazıyorum. Kimse kusura bakmasın lütfen.

Bir önceki [yazımda](/development/kotlin/2017/01/28/Neden-Kotlin.html) Kotlin'in **statik** 'type' sistemli ve **güvenli** bir programlama dili olduğunu söylemiştim. Bu yazıda, ne demek istediğimi daha detaylı bir şekilde açıklamaya çalışacağım.

## kotlin.Any ve Sınıf Hiyerarşisi

Kotlin'de class hiyerarşinin en tepesinde **Any** vardır. Hiçbir sınıftan türemeyen (super class'ı olmayan) bir sınıf oluşturursanız bu sınıfın super class'ı otomatik olarak _Any_ olur. Bire bir aynısı olmasa da _Any_'yi java'daki **Object** sınıfı olarak düşünebiliriz. 

![Any](/assets/kotlin_types/any_class_diagram.png){: .center-image }

Şekildeki gibi bir sınıf hiyerarşisini aşağıdaki Kotlin sınıfları ile oluşturabiliriz.

{% highlight kotlin %}
open class View(var position: Vector3)

class ImageView(position: Vector3) : View(position) {
    fun setDrawable(drawable: Drawable) {}
}

class TextView(position: Vector3) : View(position) {
    fun setText(s: String) {}
}
{% endhighlight %}

_ImageView_ ve _TextView_ sınıflarını _View_ sınıfından türetebilmek için _View_ sınıfını **open class** olarak tanımlamamız gerekir çünkü Kotlin'de tüm sınıflar default olarak **final**'dır. Aynı Java'da olduğu gibi Kotlin type sistemi de _super-type/sub-type_ ilişkisini kontrol eder. Super-type bir değişkene sub-type bir değer atanabilir fakat bunun tersi mümkün değildir.

{% highlight kotlin %}
val view : View = ImageView(Vector3(1,1,1)) // ok
val anotherImageView : ImageView = view // ERROR
{% endhighlight %}

## kotlin.Unit

Kotlin __expression-oriented__ bir programlama dili oldugu için her statement bir değer döndürmek zorundadır. Örneğin __if__ ve __when__ birer expression'dır.

{% highlight kotlin %}
val max = if (a > b) a else b

val dayString = when(dayOfWeek) {
    1 -> "Pazartesi"
    2 -> "Sali"
    3 -> "Carsamba"
    4 -> "Persembe"
    5 -> "Cuma"
    6 -> "Cumartesi"
    7 -> "Pazar"
    else -> "N/A"
}
{% endhighlight %}

Kotlin bu yönüyle Java'dan ayrılır, functional programlama dillerine yakınlaşır. If statement'ların java'da bir değer döndürmediklerini zaten biliyoruz. Yukarıdaki Kotlin __if__ statement örneğini Java'da yazmaya kalkarsak derleyici aşağıdaki gibi bir hata verecektir.

![java if statement](/assets/kotlin_types/if_java.png){: .center-image }

Java'daki __void__ 'un karşılığı Kotlin'de __Unit__'tir. Geriye birşey döndürme ihtiyacı olmayan fonksiyonların (sadece side effect'i olan fonksiyonlar) __return__ tipi __Unit__ olur.

{% highlight kotlin %}
fun printMessage(message: String?) : Unit {
    System.out.println(message)
}
{% endhighlight %}

Fonksiyonları tanımlarken Unit return tipini yazmak zorunda değiliz. Yukarıdaki __printMessage__ fonksiyonunu _Unit_ kullanmadan da yazabiliriz.

{% highlight kotlin %}
fun printMessage(message: String?) {
    System.out.println(message)
}
{% endhighlight %}

## kotlin.Nothing

Kotlin standart library içerisinde __Nothing__ diye de bir sınıf var. Bu özel sınıf tüm sınıf hiyerarşinin en altında yer alıyor. Eğer bir fonksiyon bir değer üretmiyorsa (sonsuz bir döngüyse ya da exception throw ediyorsa) return tipi olarak _Nothing_ kullanılabilir. Bu sayede IDE'ler ya da derleyiciler __Nothing__ dönen fonsiyonlardan sonra çalışacak kodlar için __unreachable code__ uyarısı yapabilirler.

{% highlight kotlin %}
fun error() : Nothing = throw RuntimeException("Crashed!")
{% endhighlight %}

## Kotlin'de Primitive Tipler Yok

Kotlin'de Java'daki _int_, _long_, _float_ gibi _primitive_ tipler yoktur. Herşey metodunu çağırabileceğimiz bir object'tir. Java'daki primitive tip olan _int_'e karşılık Kotlin'de _Int_ vardır ve _Int_ super-type'ı _Any_ olan bir sınıftır. Primitive tiplerin olmamasi memory kulanimi açısından problem olabilir diye endişelenmeye gerek yok çünkü Kotlin Compiler, Int (ve diğer built-in tipler Long, Float, Double, Byte ...) için gerekli optimizasyonunu yapmaktadır. Örneğin aşağıdaki kotlin ve java kodları derlendiğinde aynı byte code oluşur.

{% highlight kotlin %}
//Kotlin
val x = 5 // compiler tipi otomatik anlar (type inference)
val y : Int = 4
val z = x + y

//Java
int x = 5;
int y = 4;
int z = x + y;
{% endhighlight %}

## Elveda NullPointerException

Kotlin'in güvenli bir programalama dili olmasını sağlayan bir diğer özelliği de tiplerin _"non null"_ ya da _"nullable"_ olarak tanımlanabilmesidir. Örneğin _Int_ ve _Int?_ farklı tiplerdir fakat _Int_'i _Int?_'ın sub-type'ı olarak düşünebiliriz.

{% highlight kotlin %}
var a : Int = 5
var b : Int? = 10
a = b // *** (ERROR) non-null bir tip olan a'ya nullable bir tip atayamayız
b = a // (OK) nullable bir tipe non-null bir tip atayabiliriz.
{% endhighlight %}

Yukaridaki View-TextView-ImageView class hiyerarşisini __nullable__ tipleri de hesaba katarsak aşağıdaki gibi çizebiliriz. 

![Any](/assets/kotlin_types/any_nullable_diagram.png){: .center-image }

Nullable ve non-null tipler, type system'e dahil olduğu için Kotlin derleyicisinin NULL güvenliğini (null safety) sağlamak için ekstra bir şey yapmasına gerek kalmıyor. 

Production ortamında oluşan Exception'lar için [bir istatistik hazırlamışlar](http://blog.takipi.com/the-top-10-exceptions-types-in-production-java-applications-based-on-1b-events/) ve bu istatistiğe göre en fazla karşılaşılan exception sizin de tahmin edebileceğiniz gibi meşhur(!) __NullPointerException__ çıkmış. Bu bilgiye nasıl ulaştıklarını sorgulamadım çünkü yıllardır benim de karşıma hata olarak çoğunlukla NPE çıktı. NPE bir programcı hatası olduğu için Kotlin derleyicisini tasarlarken programcıların bu hataya tekrar tekrar düşmemeleri için __nullable__ ve __non null__ tipleri birbirinden ayırmışlar.

Nullable değişkenlerin metodlarına ve property'lerine güvenli bir şekilde (NPE'siz) erişmek için __safe call operator__  '__?.__') kullanabiliriz.

{% highlight kotlin %}
val tv : TextView = TextView(Vector3(1,1,1))
val s1 = tv.getText()

val nullableTv : TextView? = null
//val s2 = nullableTv.getText() --> (ERROR) nullable textview'in metodunu doğrudan çağırmamıza derleyici izin vermez
val s2 = nullableTv?.getText()
{% endhighlight %}

Güvenli erişimler zincir halinde de kullanılabilir. Örneğin:

{% highlight kotlin %}
val tv : TextView? = TextView(Vector3(1,1,1))
val textLength = tv?.getText()?.length
{% endhighlight %}

Yukaridaki örnekte:

* _textLength_ değişkeninin "inferred" tipi __Int?__'dir. 
* _tv_ değişkeni null referans içeriyorsa _getText()_ metodu çağırılmaz ve _textLength_ null olur. 
* _tv_ null değilse ve  _getText()_ metodu null dışında bir değer dönerse _textLength_'e Int tipinde bir değer atanır. 

__textLength__ değişkeninin hiçbir şekilde null olmasını istemiyorsak __Elvis Operator__ '__?:__' kullanabiliriz. 

{% highlight kotlin %}
var nonNullTextLength = tv?.getText()?.length ?: 0
{% endhighlight %}

Yukarıdakı örnekte _nonNullTextLength_ değişkeninin _inferred_ tipi Int (non null)'tir. Elvis operatörü solundaki expression (_tv?.getText()?.length_) değeri null'dan farklıysa solundaki yoksa sağındaki expression değerini döner.

Peki Kotlin'de hiç mi NullPointerException olmaz? İstersek olur. NPE'siz yapamam diyenler için __!!__ operatörü var. Aşağıdaki örnek çalıştırılırsa _kotlin.KotlinNullPointerException_ oluşur.

{% highlight kotlin %}
val nullableTv : TextView? = null
val s = nullableTv!!.getText()
{% endhighlight %}

NPE ile karşılaşmanın başka yolları da var. Aşağıdaki _npe()_ fonksiyonu çağırıldığında NullPointerException fırlatır.

{% highlight kotlin %}
fun npe() {
    throw NullPointerException("npe forever")
}
{% endhighlight %}

Ayrıca java dünyasından çağırdığımız fonksiyonların da NullPointerException fırlatma potansiyeli olduğunu unutmamak gerekir.