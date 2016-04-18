---
layout: post
title: Observer Pattern and Lapsed Listener Problem
categories: development general
---
We are using **Model View Controller** (MVC) architectural pattern to build mobile apps and games. MVC helps us decouple views from the rest of the application, so we can change the user interface without changing any model or logic in our app. 

How do we update the View (user interface) when a Model (data) is changed? The View can query the model periodically and if something has changed updates itself. But it will consume lots of cpu cycles needlessly if the model is not updated very often. A better solution will be registering for model update events and being notified whenever the model is updated. 

So the question is "How can an object (the model in our case) notify other objects of state changes without being dependent on their classes?". The answer is the  [Observer design pattern](https://en.wikipedia.org/wiki/Observer_pattern). 

This post will spot light on potential problems that may arise while implementing Observer pattern in your software projects.

The example I am going to use is a real life scenario taken from one of our multiplayer turn-based games. Every player in the game should make their moves in a certain amount of time (20 seconds for example). When turn changes a new timer is started for current player and the UI is updated every second to display remaining seconds. 

![Observer Pattern](/assets/observer_pattern/observer_pattern.png)

Below is a simplified version written in Java.

**CountDownObserver** is the interface that must be implemented if a class wants to be notified about countdown events.

<script src="https://gist.github.com/ilkinulas/82afc373f87f69b649aa29cc9fd9afdf.js"></script>

**CountDownTimer** is our **subject**. It calculates the elapsed and remaining times and notifies observers every second.

<script src="https://gist.github.com/ilkinulas/f6475c34ae0f26505ebe7cd47a6179e3.js"></script>

**CountDownTimerView** is the observer. It implements CountDownObserver interface. Its responsibility is to render the view.

<script src="https://gist.github.com/ilkinulas/0c5c8dd1e2c4dbfa6574d7f389b5f2a5.js"></script>

CountDownTimer (the subject) knows nothing about the views that are displaying remaining seconds. Before getting notified about countdown changes, views (the observers) must register themselves to the CountDownTimer.

### So what can possibly go wrong?

In a runtime where memory is managed automatically like Java or C#, the **subject holds a strong reference** to the observers, keeping them alive. The strong reference from the subject (the observed object) to the observer, prevents the observer (and any objects it references) from being **garbage collected** until the observer is unregistered. A **memory leak** happens, if the observer fails to unregister from the subject when it no longer needs to receive notifications. In our countdown timer example if observers do not call unregister when they become invisible, they will receive notifications and waste CPU cycles updating invisible UI elements. This issue is called [Lapsed Listener Problem](https://en.wikipedia.org/wiki/Lapsed_listener_problem).

If you implement the observer pattern in C++ (no garbage collection, memory is managed manually) and you call **delete** on one of the observers without unregistering it from the subject then you are in trouble. Now you have a dangling pointer that points to invalid data. Whenever the subject notifies its observers a **segmentation fault** will be raised. 

#### Always unregister your listeners unless they are interested in receiving notifications.

Another approach for dealing with lapsed listeners (observers) is to use [Weak References](https://docs.oracle.com/javase/7/docs/api/java/lang/ref/WeakReference.html).

The below implementation of CountDownTimer uses weak references to keep track of registered observers. If the only references to an instance are weak references, then the instance becomes a candidate for Garbage Collection. In contrast to strong references, weak references don't prevent garbage collector to finilize the instance and reclaim its memory.

<script src="https://gist.github.com/ilkinulas/f7ef529cc07a6b3a867f1a8f04db0213.js"></script>

Implementing the observer pattern by using weak references requires one more step. Now we have to find an object that has the **same lifecycle as the observers** and have hold a *strong reference to the observer*. When the strong reference holder object is garbage collected, our observer will be a eligible for garbage collection.

There is one more thing to notice; since the observers are stored in a LinkedList, registering and unregistering observers is **not thread-safe**. Most GUI implementations (and game engines) run on single thread, so we don't think of multithreading in this GUI example.

