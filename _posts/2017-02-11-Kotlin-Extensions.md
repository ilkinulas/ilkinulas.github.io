---
layout: post
title: Kotlin Extensions
categories: development kotlin
---
C# programlama dilinden tanıdığımız [Extension Method'ları](https://msdn.microsoft.com/library/bb383977.aspx), Kotlin de [destekliyor](https://kotlinlang.org/docs/reference/extensions.html). Extension'lar sayesinde  (1) yeni bir türetilmiş tip yaratmadan (extend etmeden) 
(2) mevcut tipi değiştirip tekrar derlemeden, mevcut tiplere yeni özellikler ekleyebiliriz.   

Aşağıdaki örnekte _java.util.Date_ sınıfı için yazılmış _isSunday()_ extension'ı var:

{% highlight java %}
fun Date.isSunday(): Boolean {
    val calendar = Calendar.getInstance()
    calendar.timeInMillis = time
    return calendar.get(Calendar.DAY_OF_WEEK) == Calendar.SUNDAY
}

fun main(args: Array<String>) {
    val Subat_4_1979 = SimpleDateFormat("dd/MM/yyyy").parse("04/02/1979")
    if (Subat_4_1979.isSunday()) {
        println("Bir pazar gunu dogmusum.")
    }
}
{% endhighlight %}

Kotlin derleyicisi bir final sınıf yaratıp bu sınıfa _java.util.Date_ tipinde parametre alan, _boolean_ tipinde dönüş değeri olan _isSunday_ adında static bir metod ekler. Java dünyasında, extension metodların karşılığı static metodları olan yardımcı (Util) sınıflardır. Yukarıdaki örneğin Java ile yazılmıs birebir karşılığı aşağıdaki _DateUtil_ sınıfıdır. 

{% highlight java %}
public class DateUtil {
    public static boolean isSunday(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTimeInMillis(date.getTime());
        return calendar.get(Calendar.DAY_OF_WEEK) == Calendar.SUNDAY;
    }

    public static void main(String[] args) {
        try {
            Date Subat_4_1979 = new SimpleDateFormat("dd/MM/yyyy").parse("04/02/1979");
            if (DateUtil.isSunday(Subat_4_1979)) {
                System.out.println("Bir pazar gunu dogmusum.");
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

Extension metodlarını kullanmak bir _best practice_ midir emin değilim. Faydalarını ve dikkat etmemiz gereken noktaları birer birer tartışalım.

### Code Completion

Static tipli programlama dilleri için geliştirilmiş IDE'ler yazılım sürecinde inanılmaz kolaylıklar sağlıyorlar. IntelliJ'de ayarlardan (Editor->General->Code Completion) _Code Completion_ özelliğini kapatıp yarım saat kod yazmayı deneyin ne demek istediğimi anlarsınız. 

_isSunday_ örneğinden gidecek olursak, extension metod yerine yardımcı sınıf ve static metod kullandığımızı düşünelim. _isSunday()_ metodunu kullanabilmek için önce bu metodun hangi sınıfta tanımlanmış olduğunu bulmamız gerekir. _DateUtil_, _DateUtils_, _DateHelper_, _Dates_ aklıma gelen birkaç sınıf ismi. Ancak doğru sınıfı bulduktan sonra IDE'nin _Code Completion_ özelliğini kullanabiliriz. 

Extension metodu kullanırsak IDE bize aşağıdaki resimdeki gibi bir kıyak yapabilir. Çünkü _isSunday_ metodu rastgele bir yardımcı sınıfın static metodu değil _Date_ sınıfının bir extension metodudur.

![intellisense](/assets/kotlin_extensions/extension_function_intellisense.png)

### Static dispatch

_java.sql.Date_, _java.util.Date_ sınıfından türetilmiş bir sınıftır. Sizce aşağıdaki kod bloğu çalıştırılırsa ekrana ne yazar?

{% highlight java %}
fun java.util.Date.hhmm(): String {
    return "java.util.Date ${SimpleDateFormat("HH:mm").format(this)}"
}

fun java.sql.Date.hhmm(): String {
    return "java.sql.Date ${SimpleDateFormat("HH:mm").format(this)}"
}

fun main(args: Array<String>) {
    val date1 : java.util.Date = java.sql.Date(System.currentTimeMillis())
    println(date1.hhmm())

    val date2 : java.sql.Date = java.sql.Date(System.currentTimeMillis())
    println(date2.hhmm())
}
{% endhighlight %}

_date1_ ve _date2_'nin runtime tipi aynı (_java.sql.Date_) olmasına rağmen yukarıdaki kodun çıktısı şöyledir:

```
java.util.Date 14:55
java.sql.Date 14:55
```

Bunun sebebi extension metodların dispatch edilmesinin dinamik değil statik olmasıdır. Bir başka deyişle derleyici, hangi extension metodu çalıştıracağına _date1_ ve _date2_'nin derleme sırasında tanımlı olan tiplerine bakarak karar verir.

### 'Member' fonksiyonlar her zaman kazanır

Mevcut sınıf metodu ile aynı isme, aynı sayıda ve tipte parametreye sahip bir extension metodu yazarsanız sizin metodunuz değil _'member'_ metod çağırılır. 

3rd party bir sınıf için extension metodu kullandığımızı düşünelim. Bu sınıfa ilerleyen versiyonlarda extension metodumuz ile aynı imzaya sahip bir metod eklenirse artık bizim metodunuz değil sınıfın gerçek metodu çağırılmaya başlanır ve bundan haberdar olmamız neredeyse mümkün değildir.

### Extension metod'ları nasıl organize etmek lazım?

Extension metod'ları ayrı bir dosyaya ya da kullanmak istediğiniz bir sınıf dosyasının içine yazabilirsiniz. Düzenli olması için benim tercihim 

- net.peakgames.extensions.libgdx
- net.peakgames.extensions.json
- net.peakgames.extensions.collections

gibi kendi package'ları altında (namespace) tanımlamak.

{% highlight java %}
@file:JvmName("LibGDXExtensions")
package net.peakgames.extensions.libgdx

import com.badlogic.gdx.scenes.scene2d.Actor

fun Actor.setOpacity(opacity: Float) {
    color.a = opacity
}
{% endhighlight %}

Yukaridaki örnekte [LibGDX](https://libgdx.badlogicgames.com/) Actor'leri için basit bir extension metod tanımı var. Bu extension'ı _net.peakgames.extensions.libgdx_ paketi altına LibGDXExtensions dosyasına yazıyoruz ve kullanmadan önce aşağıdaki gibi _import_ etmemiz gerekiyor:

{% highlight java %}
import net.peakgames.extensions.libgdx.opacity
...
actor.opacity(0.3f)
{% endhighlight %}

_LibGDXExtensions_ dosyasının başına yazdığımız __@file:JvmName__ annotation, Kotlin derleyicisinin LibGDX extention'ları için oluşturacağı sınıfın ismini belirtmek için kullanılıyor. _@file:JvmName_ belirtmeseydik Kotlin __LibgdxExtensionsKt__ adında bir sınıf oluşturacaktı. Yukarıda yazdığımız _opacity_ metodunu Java'dan aşağıdaki gibi çağırabiliriz.

{% highlight java %}
import net.peakgames.extensions.libgdx.LibGDXExtensions;
...
LibGDXExtensions.opacity(actor, 0.3f);
{% endhighlight %}

Sonuç olarak dikkatli ve abartmadan kullanıldığında Extension metodlar, kodun daha okunaklı olmasına yardımcı oluyorlar. Şunu hiçbir zaman aklımızdan çıkartmamamız lazım:

>With Great Power Comes Great Responsibility