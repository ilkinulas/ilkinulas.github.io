---
layout: post
title: Effectively Final Variables in Java	
categories: programming java
---
Java 8 introduced a new term : **effectively final** variables.  A variable which is **not declared as final** but whose value is never changed after initialization is effectively final. 

{% highlight java %}
for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        System.out.println("i = " + i); // Does not compile!
    }).start();
}
{% endhighlight %}

The above code does not compile, the java compiler gives the below error message for variable **i**.

{% highlight java %}
Error:(50, 45) java: local variables referenced from a lambda expression must be final or effectively final
{% endhighlight %}

To fix the compile error, loop variable i, which is not final can be assigned to an effectively final variable:

{% highlight java %}
for (int i = 0; i < 10; i++) {
    int counter = i;
    new Thread(() -> {
        System.out.println("i = " + counter);
    }).start();
}
{% endhighlight %}

Java 8 compiler can detect that the variable __counter__ remains unchanged and we can use a non-final local variable inside a lambda expression. If the value of the captured variable changes the compiler gives the same error as the above sample.

{% highlight java %}
for (int i = 0; i < 10; i++) {
    int counter = i;
    new Thread(() -> {
        System.out.println("i = " + counter); //counter value changes, does not compile!
    }).start();
    counter++;
}
{% endhighlight %}