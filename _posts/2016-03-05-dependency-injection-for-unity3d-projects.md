---
layout: post
title: Dependency Injection For Unity3D Projects (StrangeIoC)
categories: programming unity
---

The terms Dependency Injection (**DI**) and Inversion of Control (**IoC**) are generally used interchangeably to describe the same design pattern. Software can be viewed as a collection of objects collaborating with each other to perform a defined task. Without dependency injection, each object is responsible for obtaining references to its collaborators and this leads to highly coupled classes and hard to test code.

The diagram below shows a simple object graph:
![Simple Dependency Graph](/assets/ioc/dependency_graph.png){: .center-image }

+ Class A depends on Class B and C (In other words, in order to work properly, an instance of Class A needs an instance of Class B and C.)
+ Class C depends on Class D and E
+ Class D depends on Class F
+ Class E depends on Class F

So how these dependencies will be solved? Don't ever think about [Singletons](https://en.wikipedia.org/wiki/Singleton_pattern). You should stop using singletons because:

+ Singletons represent a **global state**. Code that uses global state is hard to test.
+ Singletons **tightly couple** your code to the exact type of the singleton object and removes the opportunity to use polymorphism to substitute with an alternative. 

The object graph above is very simplified. In real projects the graph will be bigger and more complex like the figure below:

![Complex Dependency Graph](/assets/ioc/dependency_graph_big.png){: .center-image }

To create and use effectively these large object graphs there is an alternative solution without using Singletons or factory classes. In this solution classes will not look for their dependencies, instead they will assume that their dependencies will be passed as constructor parameters or setter method parameters. In other words **dependencies are not pulled, they are pushed**. This is why dependency injection is also known as *Inversion Of Control*.

Injecting dependencies to classes can be done manually but if your project is big you need to write a lot of object creation and binding code yourself. So I recommend using an IoC container/framework. For our mobile Unity3D games we are using [StrangeIoC](http://strangeioc.github.io/strangeioc/). It makes our code decoupled, testable and open for extension. 

The biggest advantage of using an IoC container like *StrangeIoC* is seperating **UnityEngine** code from the rest of the application. This seperation improves portability and makes the application easily testable.

StrangeIoC has a very good documentation, [The Big Strange How To](http://strangeioc.github.io/strangeioc/TheBigStrangeHowTo.html). This post will not repeat the concepts described in the documentation. Instead I will show you how StrangeIoC helps you separate UI code from the rest of the application. Supposing that you have a game using StrangeIoC, let's look at what happens when a user clicks on the login button in your game scene.

![MVCS](/assets/ioc/MVCS.png){: .center-image }

1. The **View** is notified by UnityEngine. Views are responsible for rendering and handling user inputs. They are coupled with the underlying rendering engine. View code depends on UnityEngine.

2. Views can directly talk to their **Mediators**. Mediators are the layer where we separate UnityEngine and our application code. View calls a public method of the Mediator and Mediator dispatches a **Signal** to trigger a **Command**. Signals are *type safe event dispatching* mechanism. StrageIoC can bind Commands to Signals and you can execute commands by dispatching signals and passing them type safe parameters.

3. One or more commands can be trigger by Signals. Commands are the place where we implement the application logic. To make its job a Command can talk to Models and Services. You can seperate complex application logic into several commands and chain those commands for ordered execution.

4. In our example Login flow the command talks to UserService and checks user credentials and if the credentials are correct fetches the user data. **Services**  generally refers to anything outside the application, such as FacebookLoginService, PersistenceService, PushNotificationService ...

5. After a successful login, Command updates the user model. **Models** represent the state of the application.

6. After doing its work the command dispatches a Signal to notify the mediator. Then the signal is cleaned up. Commands do not talk directly to mediators. They communicate through the signal dispatching mechanism.

7. Finally, the mediator updates the view.

There is a lot of indirection in this simple scenerio. You can implement the same login scenario with a single View, a Model and a Service. But **the payoff for this indirection is well structured, clean and testable code**.

This was the third post in [Testing Unity Projects](/programming/unity/2016/02/20/testing-unity-projects.html) series. The next post will be about **Integration Testing** in Unity3D. By then keep calm and don't use singletons.