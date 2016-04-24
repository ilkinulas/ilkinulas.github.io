---
layout: post
title: Finite State Machines
categories: development general
---

Applications written in Object Oriented Programming (OOP) languages have states and the logic is highly dependent on the current state of the application. And games are no exception. For example, rendering and animation code might change depending on whether our hero character is currently running, jumping or attacking an enemy. The "brain" that controls the enemies in the game might  be in different states such as "chase player", "dead", "flee from player". States are everywhere, we need to implement them carefully unless we want to end up with a system full of **if and switch statements**.

This post is not about the theory and science under the finite state automatas.
In this post our task is to implement an endless runner game. The player runs, jumps over obstracles, slides under obstacles and fires bullets.

Let's start with implementing the RUNNING and JUMPING behaviours. Our first attempt will be implementing input handling in a single method called _HandleInput_.

{% highlight c# %}
public void HandleInput(Input input) {
	if (input == Input.KEY_RIGHT) {
		PlayAnimation (State.RUNNING);
	} else if (input == Input.KEY_UP) {
		PlayAnimation (State.JUMPING);
		velocity.y = JUMP_VELOCITY;
	}
}
{% endhighlight %}

There are two issues with this implementation. While RUNNING, if the user presses right-arrow key we are starting the running animation again although it is playing. If the user presses the up-arrow key and never releases, the player will be in JUMPING state and fly in the air forever. So we have to check the current state of the player when an input is received. We can fix these issues by checking the running and jumping states with two boolean flags _isRunning_ and _isJumping_ respectively.

{% highlight c# %}
public void HandleInput(Input input) {
	if (input == Input.KEY_RIGHT) {
		if ( ! isRunning ) {
			PlayAnimation (State.RUNNING);
			isRunning = true;
		}
	} else if (input == Input.KEY_UP) {
		if ( ! isJumping) {
			PlayAnimation (State.JUMPING);
			velocity.y = JUMP_VELOCITY;
			isJumping = true;
		}
	}
}
{% endhighlight %}

Let's add double jump behaviour to the player. While the player is jumping, if we press the up-arrow key player jumps higher. But a third up-arrow key press should not make the player jump again.

{% highlight c# %}
public void HandleInput(Input input) {
	if (input == Input.KEY_RIGHT) {
		if ( ! isRunning ) {
			PlayAnimation (State.RUNNING);
			isRunning = true;
		}
	} else if (input == Input.KEY_UP) {
		if ( isJumping ) {
			isDoubleJumping = true;
		} else {
			if (! isDoubleJumping) {
				isJumping = true;
			}
		}
		if ( ! isDoubleJumping ||  ! isJumping) {
			PlayAnimation (State.JUMPING);
			velocity.y = JUMP_VELOCITY;			
		}
	}
}
{% endhighlight %}

We have introduced a third boolean flag to handle double jump state. These flags can be replaced with enums but the complexity of the HandleInput method does not change much. We are not done yet, we still need to implement SLIDE and FIRE behaviours. Player can only fire while he is running and he cannot start sliding unless he is running. Before adding these two behaviours we need to step back and revise our first solution. Every time we add a new behaviour to the player we are changing the implementation of previous behaviours. We are adding flags which is not easy to maintain if there are lots of player behaviours.

To make our code more readable and to easily add new behavious without modifying existing behaviours we can use a finite state machine as seen in the picture below.

![Game FSM](/assets/state_machines/simple_fsm.jpg)

Each circle represents a _state_. Each arrow in the picture is a _state transition_. A state transition is triggered with an _event_. For example, while the state is JUMP, pressing up key triggers a PRESS_UP event and changes the state to DOUBLE JUMP. A state transition can contain **Actions**. Actions are code blocks executed on each transition. Here is how it looks if we implement input handling with a state machine class.
(The implementation details of the FiniteStateMachine class will be the subject of another blog post.)

{% highlight c# %}
private FiniteStateMachine fsm = new FiniteStateMachine ();

public void SetupStateMachine() {			
	fsm.AddTransition (State.RUNNING, Event.PRESS_UP, State.JUMPING, JumpingAction);
	fsm.AddTransition (State.RUNNING, Event.PRESS_DOWN, State.SLIDING, SlidingAction);
	fsm.AddTransition (State.RUNNING, Event.PRESS_SPACE, State.FIRING, FiringAction);
	fsm.AddTransition (State.JUMPING, Event.PRESS_UP, State.DOUBLE_JUMPING, DoubleJumpingAction);
	fsm.SetInitialState (State.RUNNING);
}

public void HandleInput (Input input) {
	switch (input) {
	case Input.KEY_DOWN:
		fsm.HandleEvent (Event.PRESS_DOWN);
		break;
	case Input.KEY_UP:
		fsm.HandleEvent (Event.PRESS_UP);
		break;
	case Input.KEY_SPACE_BAR:
		fsm.HandleEvent (Event.PRESS_SPACE);
		break;
	default:
		break;
		
	}
}
{% endhighlight %}

Adding a new behaviour to the player is straightforward. Just define a new State, add a new state transition to the new state, and implement the action that will be executed while switching to the new state. 

State machines are also useful if you want to visualize the states of the program.  [Graphviz](http://www.graphviz.org/) is an open source graph visualization software. Since finite state machines are directed graphs we can visualize them with graphviz. Graphviz reads a **DOT** file and generates an image of the graph in desired formats (png, pdf,...). DOT is a plain text graph description language. It is a simple way of describing graphs that both humans and computer programs can read. Here is our state machine description in DOT file format. 

{% highlight javascript %}
digraph { 

  node [shape=circle,fontsize=12,fixedsize=true,width=0.8]; 
  edge [fontsize=6]; 
  rankdir=LR;

  "RUN" [shape="doublecircle" color="blue"];
  "RUN" -> "JUMP" [label="PRESS\nUP"]
  "JUMP" -> "RUN" [label="JUMP\nEND"]
  "JUMP" -> "DOUBLE\nJUMP" [label="PRESS\nUP"]
  "DOUBLE\nJUMP" -> "JUMP" [label="DOUBLE JUMP\nEND"]
  "RUN" -> "SLIDE" [label="PRESS\nDOWN"]
  "SLIDE" -> "RUN" [label="SLIDE\nEND"]  
  "RUN" -> "FIRE" [label="PRESS\nSPACE"]
  "FIRE" -> "RUN" [label="FIRE\nEND"]
}
{% endhighlight %}

The resulting image of the state machine is much more beautiful than the one I draw on paper.

![Graphviz Generated FSM](/assets/state_machines/graphviz_generated_fsm.png)

As the game logic gets complicated the state machine that represents the game also gets complicated. The below picture is a visual representation of the fsm that we use in our game [Gin Rummy Plus](https://play.google.com/store/apps/details?id=net.peakgames.ginrummyplus).

![Gin Rummy Plus FSM](/assets/state_machines/ginRummyFsm.png){: .center-image }

Finally here are properties of finite state machines:

* They are deterministic. An event only causes one state transition. In other words [current state, event, next state] triple is always unique in the state machine.
* The state machine can only be in one state at a time.
* The state machine has fixed number states.
* Each state is connected to a state. 

So what do you think? Does state machines make your code organized and make your code easily adapt  to changes? If so please leave a comment and share your experience.
