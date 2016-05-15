---
layout: post
title: Filtering Unity Logs With Logcat (Android)
categories: development unity
---
Android application logs can be viewed with the **adb logcat** command. For more [adb commands](/android/2016/02/03/adb-tips.html) you can read [this post](/android/2016/02/03/adb-tips.html). **UnityEngine.Debug.Log(string message)** writes message to the console with the stacktrace information that follows immediately after any custom message you specify. IMHO these extra stacktrace information is not needed - does not makes our lives easier while debuging unity games on an android device.

Debug logs should help find the bugs and tell everything important. Extra data added by UnityEngine to the logs makes the logs show twice less information and increases visual clutter. Removing noise from the logs and making important lines easily recognizable saves your precious time. [This old post](/programming/2016/02/05/sublime-text-syntax-highlighting.html) tells how we use syntax highlighting in Sublime Text editor to make our logs more eye-friendly.

Below is the screenshot of logcat output of a Unity3D game running on an android emulator (API level 23). Can you see the needless spam? 

![Unity Logcat Verbose](/assets/logcat_unity/logcat_unity_verbose.png){: .center-image }


We should remove unnecessary information lines so we can focus on the lines that are more meaningful for us. Below python script reads the logcat output and filters out Unity Engine logs. So we can see our log messages more clearly.

<script src="https://gist.github.com/ilkinulas/2789aabaf89f38df62bd12e2e6990469.js"></script>

Here is a screenshot of the same logcat output processed with the above script:

![Unity Logcat Brief](/assets/logcat_unity/logcat_unity_brief.png){: .center-image }

Warnings and errors are printed with a different color. This makes it very easy to see these lines among hundreds of lines. Exceptions are also marked with a different color. Script can be run with the following shell command on MAC and Linux platforms. 

{% highlight csharp %}
adb logcat -v time | python logcat_unity.py
{% endhighlight %}

Try it and you're welcome to add new features to [the script](https://gist.github.com/ilkinulas/2789aabaf89f38df62bd12e2e6990469).