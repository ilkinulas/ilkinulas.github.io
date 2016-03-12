---
layout: post
title: Writing Integration Tests For Unity3D Projects
categories: programming unity
---
This is the 4th and the last post of series, [Testing Unity Projects](/programming/unity/2016/02/20/testing-unity-projects.html). After being sure that every unit (class) works as expected, we should also check how classes collaborate with each other by writing integration tests.


![2 Unit tests, 0 Integration Tests](/assets/integration_tests/no_integration_test.gif){: .center-image }


The scope of integration tests is wider than the scope of unit tests. Unlike unit tests integration tests might depend on other outside systems, in our case UnityEngine. Integration tests cover whole applications, and they require much more effort to put together. 

Unity Test Tools package comes with an [Integration Test Framework](https://bitbucket.org/Unity-Technologies/unitytesttools/wiki/IntegrationTestsRunner). You can automate the testing of your assets and the interaction between your assets within a scene by writing integration tests.

There are two ways of writing integration tests:

1. **Manual** : You setup your scene manually and use assertion components.
2. **Dynamic** : Integration tests are setup from code (dynamically)

We are using dynamic integration tests, and we setup our tests fully from code which is more flexible than manually setting up scenes for integration tests.

Integration test framework automatically finds classses annotated with **IntegrationTest.DynamicTest** attribute and if the scene name matches the current scene, the test is run in the scene. There are other attributes that can be used for dynamic tests. Check the [documentation](https://bitbucket.org/Unity-Technologies/unitytesttools/wiki/IntegrationTestsRunner) for a full list of attributes.

{% highlight csharp %}
[IntegrationTest.DynamicTest("GameScene")]
[IntegrationTest.ExcludePlatformAttribute(RuntimePlatform.Android, RuntimePlatform.IPhonePlayer)]
[IntegrationTest.Timeout(60)]
public class FacebookLoginTest: IntegrationTestBase {
	...
	...
}
{% endhighlight %}

The dynamic tests are marked with **HideFlag.DontSave** and will not be saved on the scene. 

Integration tests can be run in batch mode. We are using the below command to run our  integration tests. The results are reported in nUnit result format.

{% highlight bash %}
/Applications/Unity-5.3.1-f1/Unity.app/Contents/MacOS/Unity \
  -batchmode \
  -projectPath $(pwd) \
  -executeMethod UnityTest.Batch.RunIntegrationTests \
  -testscenes=GameScene \
  -resultsFileDirectory=$(pwd)
{% endhighlight %}

Although the integration test framework is designed to run on a separate test scene, we are writing our dynamic integration tests for our real game scene. This way we can write end-to-end tests. We can simulate a user playing the game and make assertions while the integration test is running in the game scene. 

This is a video recorded while one of our integration tests is running. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/y4u4IJJ06sQ" frameborder="0" allowfullscreen></iframe>

