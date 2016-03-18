---
layout: post
title: Unity UI Drag Threshold 
categories: programming unity
---

Devices like Samsung Galaxy S5 or Samsung Galaxy S6 are high DPI devices. So what is DPI? DPI is an acronym for Dots Per Inch and it refers to the number device pixels per inch. If the DPI is higher,  pixel size is smaller.

If you have a unity project running on these High DPI devices you may have issues pressing buttons because very small movements in finger touch can be interpreted as scrolling rather than a simple touch.

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