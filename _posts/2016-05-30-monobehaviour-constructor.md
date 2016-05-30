---
layout: post
title: Don't Use Constructors To Initialize Monobehaviours.
categories: development unity
---

Monobehaviours are building blocks of developing games in unity. You can create components and assign them to gameobjects to control their behaviour. You can think of components as scripts derived from monobehaviour base class.


Monobehaviours have special methods that are called by the unity engine. Unity documentation says:

* **Awake:** This function is always called before any Start functions and also just after a prefab is instantiated. (If a GameObject is inactive during start up Awake is not called until it is made active.)
* **Start:** Start is called before the first frame update only if the script instance is enabled.
* **Update:** Update is called once per frame. It is the main workhorse function for frame updates.

**Awake** and **Start**  methods are used to initialize components before they become active in the scene.

Experienced C# developers know that classes have constructors and the constructor is called once for every instance created. We put initialization code (e.g., setting default values) in the constructor and we should avoid adding application logic to constructors.

Since Monobehaviours are classes, they also have constructors. If you do not define one the compiler will generate a default constructor for you. **You should avoid using constructors to initialize monobehaviours.** Monobehaviours should be initialized with special methods called Awake or Start.

To see in what order the constructor, Awake and Start methods are called, create a test scene and add an empty game object. Attach the below script to the newly created gameobject.

{% highlight csharp %}
using UnityEngine;

public class ConstructorTest : MonoBehaviour {

	public ConstructorTest() {
		Debug.Log ("Constructor is called");
	}

	void Awake () {
		Debug.Log ("Awake is called");
	}

	void Start () {
		Debug.Log ("Start is called");
	}

	void Update () {		
	}
}
{% endhighlight %}

When you hit play you can see the below lines printed in the console. (The constructor is called **twice**.)
<pre>
Constructor is called
Constructor is called
Awake is called
Start is called
</pre>

More interestingly when you hit play again to exit the play mode you can see that the constructor is called again:
<pre>
Constructor is called
</pre>

Depending on what you do in the constructor you can see unexpected results if you execute code in the constructor.

Another thing to note is that constructors of monobehaviours are not called on the main thread. Change the constructor as below:
{% highlight csharp %}
public ConstructorTest() {
	Debug.Log ("Constructor is called" + this.gameObject);
}
{% endhighlight %}
And when you hit play you can see in the console that unity is complaining about using constructors:
<pre>
ArgumentException: get_gameObject can only be called from the main thread.
Constructors and field initializers will be executed from the loading thread when loading a scene.
Don't use this function in the constructor or field initializers, instead move initialization code to the Awake or Start function.
ConstructorTest..ctor () (at Assets/ConstructorTest.cs:6)
</pre>

