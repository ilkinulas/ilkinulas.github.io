---
layout: post
title: The Joy Of Mocking With NSubstitute
categories: programming unity
---

While writing unit test, the class under test may have dependencies. And these dependencies may be impossible to instantiate in a test fixture or they may be very difficult to setup. To make unit testing simpler and to focus on the code being tested, not on the behavior or state of external dependencies, [software craftsmen](http://manifesto.softwarecraftsmanship.org/) use the technique called **mocking**.

In mocking, the dependencies are replaced by artificial objects that simulate the behavior of the real ones. Mock objects have the same interface as the real objects they mimic, allowing a client object to remain unaware of whether it is using a real object or a mock object. The words **mock** and **substitute** are used interchangeably throughout this document.

For every programming language I know, there exists a mocking framework that makes it easy to mock real objects. This post will show how to create and use mock objects with [NSubstitute](http://nsubstitute.github.io/) for C# based Unity3D projects.

**NSubstitute** library is shipped with [Unity Test Tools](https://www.assetstore.unity3d.com/en/#!/content/13802) package. 

### Creating Substitutes
First you should include NSubstitute namespace in your test class.
{% highlight csharp %}
using NSubstitute
{% endhighlight %}

Then create Substitutes for interfaces like below.
{% highlight csharp %}
IGameView view = Substitute.For<IGameView> ();
IAnalyticsService analyticsService = Substitute.For<IAnalyticsService>();
IHttpClient httpClient = Substitute.For<IHttpClient>();
{% endhighlight %}

Although NSubstitute library is able to substitute real classes, you should always create substitutes for interfaces. Writing code against interfaces makes your design more testable which implicitly makes your design good.

A good unit test generally has three parts: **Setup-Call-Assert**

+ **Setup** : Create substitutes and setup return values.
+ **Call** : Call the code under test. 
+ **Assert** : Make assertions that specify the pass criteria for the test.

### Setup Substitute Return Values

#### Methods without parameters
{% highlight csharp %}
InputInterface input = Substitute.For<InputInterface> ();
input.MousePosition ().Returns (Vector3.zero);

Vector3 mousePos = input.MousePosition (); // mousePos is (0,0,0)
{% endhighlight %}

#### Methods with parameters

In the example below, if the __accountManager.GetAccount__ method is called with parameter __"ilkin"__ a new account is returned to caller. If method parameter is __"missingAccountName"__ an exception is thrown.

{% highlight csharp %}
IAccountManager accountManager = Substitute.For<IAccountManager> ();
Account account = new Account ();
account.Name = "ilkin";
accountManager.GetAccount (Arg.Is ("ilkin")).Returns (account);
accountManager.GetAccount ("missingAccountName").Returns(
	x => {throw new Exception("Account not found");}
);
{% endhighlight %}

Return values for properties is set in the same way as for a method using ***"Returns()"***

You can return values for any method parameters with **"ReturnsForAnyArgs()"** method:

{% highlight csharp %}
[Test]
public void test() {
	IFortuneTeller fortuneTeller = Substitute.For<IFortuneTeller> ();
	fortuneTeller.tellMe ("").ReturnsForAnyArgs ("NO");

	Assert.AreEqual ("NO", fortuneTeller.tellMe ("Are you telling the truth?"));
}
...
public interface IFortuneTeller {
	string tellMe(string question);
}
{% endhighlight %}

#### Void calls and 'When...Do'

We can not use **Returns()** on void methods because they can't return anything :) To perform an action when a void method is called you should use the **When..Do** method chain. Here is an example:

{% highlight csharp %}
[Test]
public void test() {
	bool isGoogleDown = false;
	OnSuccess successCalback = s => {
		isGoogleDown = false;
	};
	OnError errorCallback = e => {
		isGoogleDown = true;
	};

	httpClient.When (
		client => client.Get ("http://www.google.com", successCalback, errorCallback)
	).Do (callInfo => errorCallback(new Exception("Google is down!")));

	httpClient.Get ("http://www.google.com", successCalback, errorCallback);

	Assert.IsTrue (isGoogleDown);	
}
...
public delegate void OnSuccess(string response);
public delegate void OnError(Exception error);

public interface IHttpClient {	
	void Get (string url, OnSuccess successCallback, OnError errorCallback);
}
{% endhighlight %}


### Checking Received Calls (Assertions)
After the substitutes are setup and the code under test is called, we need to check if the interaction between the substitutes and code under test is as expected. To check that a specific call has been received by a substitute we use **Received**.

{% highlight csharp %}
httpClient.Received ().Get ("http://www.google.com", successCalback, errorCallback);

//Check Get is called a specific number of times
httpClient.Received (1).Get ("http://www.google.com", successCalback, errorCallback);

//Check Get is not called.
httpClient.DidNotReceive (1).Get ("http://www.google.com", successCalback, errorCallback);
{% endhighlight %}


For further reading you should check the [official NSubstitute Documentation](http://nsubstitute.github.io/help.html).

There is also a blog post by Unity Technologies which explains testing with mocks in Unity3D. [UNIT TESTING AT THE SPEED OF LIGHT WITH UNITY TEST TOOLS](http://blogs.unity3d.com/2014/07/28/unit-testing-at-the-speed-of-light-with-unity-test-tools/)

In the next post we are going to dive into Integration Testing. By then keep calm and enjoy unit testing.