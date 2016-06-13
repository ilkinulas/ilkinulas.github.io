---
layout: post
title: Reverse Engineering Apk Files
categories: android
---
The site **Appbrain**  has a [develoepr tools stats](http://www.appbrain.com/stats/libraries/dev) page that lists top used SDKs in android applications and games. Do you ever wonder how can we see which sdk an app uses?  You can download an apk, open it and apply reverse engineering technics to convert the machine readable binary to human readable files.

The first step is obtaining the apk. There are several ways to get the apk of the app. You can either pull it from the device (I am going the tell how in a minute) or download it from web sites like [apkmirror.com](http://www.apkmirror.com/)

The command below lists all the installed packages on the connected android device or emulator. The **-f** option is used to see the associated apk file of the package.

{% highlight bash %}
adb shell pm list packages -f
{% endhighlight %}

The output of this command will look like this depending on the packages on your device:

<pre>
package:/system/app/Apollo.apk=com.andrew.apollo
package:/system/app/GoogleEars.apk=com.google.android.ears
package:/system/priv-app/VoiceDialer.apk=com.android.voicedialer
...
package:/data/app/com.supercell.clashroyale-1.apk=com.supercell.clashroyale
...
</pre>

Once you see the path of the apk file, you can **pull** it to your computer with the **adb pull** command:

{% highlight bash %}
adb pull /data/app/com.supercell.clashroyale-1.apk
ls -l
total 186744
-rw-r--r--  1 ilkinulas  admin  95612682 Jun 13 08:42 com.supercell.clashroyale-1.ap
{% endhighlight %}

The next step is to use [Apktool](http://ibotpeaches.github.io/Apktool/) to decode resources to nearly original form. After downloading and installing **apktool** run the command **apktool decode** :

{% highlight bash %}
apktool decode com.supercell.clashroyale-1.apk
I: Using Apktool 2.0.2 on com.supercell.clashroyale-1.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/ilkinulas/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...

ls -ltr
total 186744
-rw-r--r--  1 ilkinulas  admin  95612682 Jun 13 08:42 com.supercell.clashroyale-1.apk
drwxr-xr-x  9 ilkinulas  admin       306 Jun 13 08:48 com.supercell.clashroyale-1
{% endhighlight %}

**apktool decode** command decodes the binary package and generates a folder with the same name of the apk file.

![Apk contents](/assets/reverse_engineer_apk/apkcontents.png){: .center-image }

We can see which 3rd party SDKs the app is using by looking at the **smali** folder. Apktool also decodes the following files:

+ AndroidManifest.xml
+ Files under the resources folder (res).
+ classes.dex 

Android's Java virtual machine implementation (aka Dalvik VM) uses dex files instead of java class files. Apktool decodes dex files to **smali** format. Smali/Baksmali is an assembler/disassembler for the dex format used by Dalvik VM. The names "Smali" and "Baksmali" are the Icelandic equivalents of "assembler" and "disassembler" respectively. If you want to see the java source code of the decoded apk file but don't understand smali syntax then you need another tool called [dex2jar](https://sourceforge.net/projects/dex2jar/). 

Below command generates a java archive from the the classes.dex file:


{% highlight bash %}
/Users/ilkinulas/bin/dex2jar-2.0/d2j-dex2jar.sh com.supercell.clashroyale-1.apk
dex2jar com.supercell.clashroyale-1.apk -> ./com.supercell.clashroyale-1-dex2jar.jar
{% endhighlight %}

You can see the generated class files by looking inside the generated jar file.

{% highlight bash %}
tar -tvf com.supercell.clashroyale-1-dex2jar.jar|more
drwxrwxrwx  0 0      0           0 Jun 13 09:19 android/
-rwxrwxrwx  0 0      0         133 Jun 13 09:19 android/UnusedStub.class
drwxrwxrwx  0 0      0           0 Jun 13 09:19 android/support/
drwxrwxrwx  0 0      0           0 Jun 13 09:19 android/support/annotation/
-rwxrwxrwx  0 0      0         448 Jun 13 09:19 android/support/annotation/AnimRes.class
-rwxrwxrwx  0 0      0         452 Jun 13 09:19 android/support/annotation/AnimatorRes.class
-rwxrwxrwx  0 0      0         447 Jun 13 09:19 android/support/annotation/AnyRes.class
...
{% endhighlight %}

The java class files are still not human readable. We need a java decompiler to convert the binary class files to readable java source files. [JD Project](http://jd.benow.ca/) comes the the rescue. 

![Java Decompiler](/assets/reverse_engineer_apk/jdgui.png){: .center-image }

The Java Decompiler generates java source files from the class files. You can now see the contents of the classes in the apk. (If the java classes are obfuscated decompiling the class files does not help much.)

If you find this guide useful or know a better way please leave a comment. 