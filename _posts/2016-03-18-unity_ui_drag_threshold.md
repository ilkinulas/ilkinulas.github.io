---
layout: post
title: Unity UI Drag Threshold 
categories: programming unity
---

If you have a unity project running on High DPI devices like Samsung Galaxy S5 or Samsung Galaxy S6, you may have issues pressing buttons because very small movements in finger touch can be interpreted as scrolling rather than a simple touch.

So what is **[DPI (Dots per inch)](https://en.wikipedia.org/wiki/Dots_per_inch)**? DPI is the number of device pixels per inch. If the DPI is higher,  pixel size is smaller. In other words there exists more pixels in a in High-DPI devices.


Whenever you add a Canvas to a Scene an EventSystem is automatically added to the hierarchy. The EventSystem component has a property called **Drag Threshold** for specifying the soft area for dragging in **pixels**. 

![Event System Component](/assets/unity_drag_threshold/event_system.png){: .center-image }

Default value of Drag Threshold is 5 pixels which is very small for high DPI devices. Below script updates Drag Threshold according to [Screen.dpi](http://docs.unity3d.com/ScriptReference/Screen-dpi.html).

{% highlight csharp %}
public class DragThresholdUtil : MonoBehaviour {
  void Start () {
    int defaultValue = EventSystem.current.pixelDragThreshold;		
    EventSystem.current.pixelDragThreshold = 
            Mathf.Max(
                 defaultValue , 
                 (int) (defaultValue * Screen.dpi / 160f));
  }
}
{% endhighlight %}

For example, "Drag Threshold" is updated to "15" pixels for Samsung Galaxy S5 which is a 480 DPI device.

You may wonder where this magical number **160** comes from. It is the accepted (1) DPI value for medium sized screen devices. 

(1) [http://developer.android.com/guide/practices/screens_support.html](http://developer.android.com/guide/practices/screens_support.html)