---
layout: post
title: Testing Unity Projects
categories: programming unity
---

Professional software developers write automated tests. It is not because they make a lot of mistakes, it is because :

* they **care** about the product they develop. 
* they want to be sure that their product **works**.
* they want their product easily **adapt changes**.
* they don't want to commit, push changes and pray. They want to know that their **changes do not break anything**.

Here is a poem which puts emphasis on the importance of writing automated tests for software projects :)

> 99 little bugs in the code <br/>
  99 little bugs in the code <br/>
  Fix one bug, compile it again <br/>
  101 little bugs in the code <br/>

This series of posts is for Unity3D game developers who wants to write automated tests for their game projects.

+ Part I    - [Unit testing and Unity Test Tools (this post)](/programming/unity/2016/02/20/testing-unity-projects.html)
+ Part II   - [Mocking with NSubstitute](/programming/unity/2016/02/25/mocking-with-nsubstitute.html)
+ Part III  - [Dependency Injection (Writing Testable Code)](/programming/unity/2016/03/05/dependency-injection-for-unity3d-projects.html)
+ Part IV   - Integration tests in Unity3D


# Part I - Unit testing in Unity3D
Writing unit tests is hard because __there is a deep synergy between testability and good design__. Michael Feathers has an [excellent talk](https://www.youtube.com/watch?v=4cVZvoFGJTU) about how testability leads us to good design.

As I write this post latest Unity3D version is [5.3.2](http://unity3d.com/get-unity/download?ref=personal). Before diving into testing you should import [Unity Test Tools](https://www.assetstore.unity3d.com/en/#!/content/13802) package (which is developed by Unity Technologies).

Unity uses [NUnit Framework](http://www.nunit.org/) for unit tests. Engineers at Unity took NUnit and wrote a UI for running tests inside the Unity editor. You can open the Editor Test Runner through menu "Window -> Editor Test Runner". 

![Editor Test Runner](/assets/testing_unity/editor_test_runner.png){: .center-image }

You should place your unit test files under the Editor folder. Editor Test Runner then finds the tests automatically and displays them in a tree structure (see picture above) where you can run tests and see test results.

**NUnit** uses custom atributes to identify tests. [Here](https://github.com/nunit/docs/wiki/Attributes) is a list of all custom attributes in the **NUnit.Framework** namespace.

#### [[Test]](https://github.com/nunit/docs/wiki/Test-Attribute)

Test attribute marks a method as a test. Only this attribute is mandatory for writing tests, because the test runner finds your tests by searching methods annotated by Test attribute.

#### [[SetUp]](https://github.com/nunit/docs/wiki/SetUp-Attribute)
Methods annotated with SetUp attribute are called once before each test case. If you have 5 methods annotated with [Test] attribute then SetUp method runs 5 times.

#### [[TearDown]](https://github.com/nunit/docs/wiki/TearDown-Attribute)
TearDown methods are called once after each test case. If SetUp method fails or throws an exception,	 TearDown method will not run.

#### [[TestFixtureSetUp]](https://github.com/nunit/docs/wiki/TestFixtureSetUp-Attribute) and [[TestFixtureTearDown]](https://github.com/nunit/docs/wiki/TestFixtureTearDown-Attribute)
These two are for one time setup and teardown.

Suppose that you have 3 test cases in a test fixture. Then the execution order of all the methods would be:
<pre>
	TestFixtureSetUp
		SetUp
			TestCase1
		TearDown			
		SetUp
			TestCase2
		TearDown
		SetUp
			TestCase2
		TearDown		
	TestFixtureTearDown
</pre>

Enough talk, time to see some code. There is a demo project called [CodeBreaker](https://github.com/ilkinulas/CodeBreaker).  At the time of writing this post the project is not complete yet. I am planning to add more tests and functionality to the demo project as I write more posts about testing unity projects.

The demo project is a simple implementation of two-player code breaking game called [Mastermind](https://en.wikipedia.org/wiki/Mastermind_(board_game)).

Below is a sample unit test class taken from the demo project that demonstrate the usage of NUnit Attributes. 

<script src="http://gist-it.appspot.com/https://github.com/ilkinulas/CodeBreaker/blob/master/Assets/Editor/UnitTest/GameLogicTest.cs?slice=1:37"></script>

This is the first post of series about testing in Unity3D projects. In the next post I am going to write about using mocks in unit tests.

Finally, while writing unit tests always keep in mind that a good unit test :

+ **runs fasts**. In a modest project there are hundreds of unit tests. If it takes 1 second to run a single test then all tests will run in more than one and a half minute and that's enough time to grap a coffee from the vending machine. You should see the result of the unit tests **instantly**.
+ should be part of your **Continious Integration** [(CI)](https://en.wikipedia.org/wiki/Continuous_integration). Tests should be run automatically on every commit and failing tests should fail the build.
+ must be **isolated**. Tests should not rely on each other.
+ must be written as carefully as production code. Developers should follow the same standard of good-design for their test code. No duplication, tests with good names, etc. And tests should be versioned as same as production code.
+ should **test only one thing** at a time. (Multiple assertions in a single unit test is fine but when a test fails it should spot the location of the problem.)
