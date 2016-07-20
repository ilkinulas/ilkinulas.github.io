---
layout: post
title: How To Customize Unity3d Hierarchy Window
categories: unity
---
One of the many things I like about Unity is its customizable Editor. Although it crashes every few hours, its visual environment reduces development time. The Unity editor is very customizable, so that you can create custom tools for your needs.

These days I am working on a [mobile card game](https://play.google.com/store/apps/details?id=net.peakgames.ginrummyplus). In the game there are four main monobehavior types: Card, Hand, Deck and Pile. A card gameobject can be:

- enabled
- disabled
- child of a Hand gameobject
- child of a Deck gameobject
- child of a Pile gameobject 

While playing the game, cards are moving between Deck, Hand and Pile. It would be very useful for tracking the cards in the editor hierarchy window. For example, by looking at the hierarchy window I want to see the number of active card gameobjects which are children of deck gameobject. The below gif is recorded from a demo app. Each time the "Deal Cards" button is pressed, the Cards in Deck are dealt to Players' Hands. You can see the numbers at the right of items changing as the hierarchy is updated. These numbers represent the number of Cards (enabled and disabled) under the specified gameobject.

![Custom Hierarchy Window](/assets/custom_hierarchy_window/custom_hierarchy_window.gif){: .center-image }

> I prefer and highly recommend changing the editor color in play mode. (I am using a shade of green for Playmode tint color)

The hierarchy window can be customized in three easy steps:

#### 1. Create A Class with an [InitializeOnLoad] Attribute
The [InitializeOnLoad]() attribute marks the class to be loaded automatically when the Unity editor is loaded (without any user action.)

{% highlight csharp %}
[InitializeOnLoad]
public class CustomHierarchyView  {
	...
}
{% endhighlight %}


#### 2. Add a static constructor
A static constructor (C#) method is guaranteed to run at most once. But note that for a class marked with the **InitializeOnLoad** attribute, the static constructor is called once for the editor load and once for the application run. 

{% highlight csharp %}
[InitializeOnLoad]
public class CustomHierarchyView  {

  static CustomHierarchyView() {		
  }

}
{% endhighlight %}

#### 3. Register to EditorApplication.hierarchyWindowItemOnGUI event
__EditorApplication.hierarchyWindowItemOnGUI__ delegate is called for every visible item in the HierarchyWindow on every OnGUI event. This is where we do the real customization work.

{% highlight csharp %}
[InitializeOnLoad]
public class CustomHierarchyView  {

  static CustomHierarchyView() {
    EditorApplication.hierarchyWindowItemOnGUI += HierarchyWindowItemOnGUI; 
  }

  static void HierarchyWindowItemOnGUI (int instanceID, Rect selectionRect) {			
    ... 
  }
	
}
{% endhighlight %}

The delegate has to parameters:

- **Instance Id** : The instance id of an object in the scene is always guaranteed to be unique. You can find a gameobject by its instance id by using [InstanceIDToObject](http://docs.unity3d.com/ScriptReference/EditorUtility.InstanceIDToObject.html) method. This method only runs in editor scripts.

- **Selection Rectangle** : Once we find the object we can customize its rendering in the editor. The selection rectangle parameter contains the information about the position and size of the list item in the hierarchy. The image on the right renders each item in the hierarchy as a rectangle. 

![Selection Rectangle](/assets/custom_hierarchy_window/side_by_side_rect.png){: .center-image }


The complete C# editor script is made available as a [gist @ github](https://gist.github.com/ilkinulas/802c993bb6bcdb3a45bfbdd01c2f3718). Use your imagination and coding skills to build a better editor for your game.