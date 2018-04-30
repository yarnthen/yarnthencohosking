---
layout: post
title: "Battle City in Unity Part 6: Enemy AI"
excerpt: Battle City in Unity Part 6
summary: This post is part 6 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}
### Skynet. Anyone?

Sorry, we are not going to create a Skynet nor a Deep Blue nor something remotely capable of coming up with tactics to flank your tank trapping you in a corner where you cannot escape. These are beyond my meager knowledge; I apologize for my incapability. That could be something of a wishlist later if I manage to gain sufficient insight on that.

Instead, we will be using the most basic form of simulating AI, by randomness.

### Heir to the Movement Script again

I mentioned in the previous post AI script is the other script which will inherit from Movement script. So let's start by creating a script called EnemyAI and update the Class declaration to inherit from Movement.

{% highlight csharp%}
public class EnemyAI : Movement {
{% endhighlight %}

### Creating the unpredictable feeling

So what can we make random about the Enemy? The most straightforward one I can think of is its direction. By randomizing its heading, we will create a sense of AI. The code is pretty simple; we need to ask the computer to pick a random direction(up, down, left, right), then move in that direction if it's selected. We will use <keyword>enum</keyword> to create the Direction. Let's put the code in a routine named <keyword>RandomDirection</keyword>.

{% highlight csharp%}
float h, v;
enum Direction { Up, Down, Left, Right };
Direction[] direction = { Direction.Up, Direction.Down, Direction.Left, Direction.Right };
public void RandomDirection()
    {
        
        Direction selection = direction[Random.Range(0, 5)];
        if (selection == Direction.Up)
        {
            v = 1;
            h = 0;
        }
        if (selection == Direction.Down)
        {
            v = -1;
            h = 0;
        }
        if (selection == Direction.Right)
        {
            v = 0;
            h = 1;
        }
        if (selection == Direction.Left)
        {
            v = 0;
            h = -1;
        }
    }
{% endhighlight %}
<div class="info">Using enum in this is up to your preferences. enum make the code longer but more readable. You can just use number 1 to 4 to represent the different direction. Of course someone else reading your code will need to know of your number assignment</div>
With that done, all we need to do is to add the same code in Player script which checks for h and v in FixedUpdate. Remember to do a GetComponent for the rigidbody 2d at the Start so that we can pass it to the coroutine as parameter.

{% highlight csharp%}
Rigidbody2D rb2d;
void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
}

void FixedUpdate()
{
    if (v != 0 && isMoving == false) StartCoroutine(MoveVertical(v, rb2d));
    else if (h != 0 && isMoving == false) StartCoroutine(MoveHorizontal(h, rb2d));
}
{% endhighlight %}


### Determine when to be unpredictable: Collision

Now we have the means to be unpredictable; the computer needs to know when it should be erratic. Looking through the gameplay of Battle City, you would have noticed that enemy tanks always changes direction when it hit an obstacle, so we could initiate a “RandomDirection” whenever we hit a barrier.

So let’s add the Monobehaviour routine <keyword>CollisionEnter2D</keyword> to the EnemyAI script. For now, we will only ask it to start the routine RandomDirection when a collision is detected.

{% highlight csharp%}
void OnCollisionEnter2D(Collision2D collision)
{
    RandomDirection();
}
{% endhighlight %}

Also, add the routine RandomDirection at the Monobehaviour routine <keyword>Start</keyword>. Adding the routine to start will ensure the EnemyAI will start the tank moving as soon as the Game Object appears in the game scene.

{% highlight csharp%}
void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
    RandomDirection();
}
{% endhighlight %}

I consolidate all the code for EnemyAI until now at the below. The code below is sufficient to make a working Enemy AI. You can try out by disabling the Player (Script) component from your guinea pig Tank and add the EnemyAI script. Press play and you can see your Enemy tank become self-aware and move by itself!

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyAI : Movement {
    Rigidbody2D rb2d;
    float h, v;
    enum Direction { Up, Down, Left, Right };
    Direction[] direction = { Direction.Up, Direction.Down, Direction.Left, Direction.Right };
    void Start()
    {
        rb2d = GetComponent<Rigidbody2D>();
        RandomDirection();
    }   
    public void RandomDirection()
    {        
        Direction selection = direction[Random.Range(0, 4)];
        if (selection == Direction.Up)
        {
            v = 1;
            h = 0;
        }
        if (selection == Direction.Down)
        {
            v = -1;
            h = 0;
        }
        if (selection == Direction.Right)
        {
            v = 0;
            h = 1;
        }
        if (selection == Direction.Left)
        {
            v = 0;
            h = -1;
        }
    }
    void OnCollisionEnter2D(Collision2D collision)
    {
        RandomDirection();
    }
    void FixedUpdate()
    {
        if (v != 0 && isMoving == false) StartCoroutine(MoveVertical(v, rb2d));
        else if (h != 0 && isMoving == false) StartCoroutine(MoveHorizontal(h, rb2d));
    }
}
{% endhighlight %}


### Determine when to be unpredictable: Random timing

We could also put another random timer for the computer to pick a moment to do an unpredictable turn; this timer should have a reasonable <keyword>cooldown period</keyword> before it can trigger another turn again so we need to have the cooldown take into account a RandomDirection cause by a collision. If the cooldown is too short, we might end up having the enemy tanks spinning around like headless chickens.

The code for this is very simple, two lines only. One at the last line of RandomDirection routine to <keyword>Invoke</keyword> RandomDirection at a random interval between 3 to 5 seconds and one at the first line of RandomDirection routine to <keyword>cancel any RandomDirection invoke</keyword>. The last line is to ask the routine to trigger itself again at an random interval between 5 to 8 seconds, and the 1st line is a means to reset the timer as an Invoke of RandomDirection will be triggered at the last occurrence of the RandomDirection routine.

{% highlight csharp%}
    public void RandomDirection()
    {
        CancelInvoke("RandomDirection");
        Direction selection = direction[Random.Range(0, 4)];
        if (selection == Direction.Up)
        {
            v = 1;
            h = 0;
        }
        if (selection == Direction.Down)
        {
            v = -1;
            h = 0;
        }
        if (selection == Direction.Right)
        {
            v = 0;
            h = 1;
        }
        if (selection == Direction.Left)
        {
            v = 0;
            h = -1;
        }
        Invoke("RandomDirection", Random.Range(3, 6));
    }
{% endhighlight %}

Now try play your game again. Skynet has risen!

### Determine when to be unpredictable: Logical unpredictability

Lastly, we will need to have the enemy to make a reasonable unpredictable(sounds contradicting) direction change. For example, an enemy tank makes its way to a location where it is surrounded on 3 sides with barriers, so the only logical option for it is to turn around and head out. But if we were simply to trigger the "RandomDirection," the tank will just be spinning in multiple directions until it gets lucky enough to turn to face its back and head out. That is not AI(Artificial Intelligence), that is AS(Artificial Stupidity)!

To prevent AS from occurring, we will need to check for the direction the tank can turn to in order not to look stupid whenever the RandomDirection routine is triggered. So the best place to put this code will be in.....the RandomDirection routine! We will be using a <keyword>Physics2D static routine</keyword> called <keyword>Linecast</keyword> which takes in 2 positions and a <keyword>LayerMask</keyword>. A line is drawn to join the 2 positions and if the line hits anything that belongs the Layer specified by the layer mask, it will return true. 

We will consolidate those position with a false LineCast into a list. Then the computer will run a <keyword>Random.Range</keyword> on the items in the list to determine the direction. 

We will need two new variables for this effort, one for the LayerMask which I will call it <keyword>blockingLayer</keyword>(that is the name they use in the unity tutorials) and one is a List containing Direction enum which I will call <keyword>lottery</keyword>.

We will <keyword>linecast 1 unit from the transform.position of the GameObject</keyword> to determine if anything is blocking their path. The code will be as below.

{% highlight csharp%}
List<Direction> lottery = new List<Direction>();
if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(1, 0), blockingLayer))
{
    lottery.Add(Direction.Right);
}
if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(-1, 0), blockingLayer))
{
    lottery.Add(Direction.Left);
}
if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, 1), blockingLayer))
{
    lottery.Add(Direction.Up);
}
if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, -1), blockingLayer))
{
    lottery.Add(Direction.Down);
}
{% endhighlight %}

For the LayerMask, we will declare it as a SerializeField variable so that the layers impacted can be chosen from the Inspector.

{% highlight csharp%}
[SerializeField]
LayerMask blockingLayer
{% endhighlight %}

Once we have the list of directions we can turn to; we will pass it to the selection variable after making a random selection.
So the code will change from <keyword>Direction selection = direction[Random.Range(0, 5)]</keyword> to <keyword>Direction selection = lottery[Random.Range(0, lottery.Count)]</keyword>. We can also remove the declaration of Direction array variable called direction.

Now we need to update the <keyword>blockingLayer</keyword> variable from the Inspector. Select the Guinea Pig tank and find the blockingLayer variable in the Enemy AI (Script) component. It should be showing Nothing. Click on it to access the drop-down to select the layers. Select Boundary(Steel, and Boundary Tilemap) and Water(Water Tilemap) as these are possible layers that will collide with the tanks. 

<div class="info">A Brick tile is an obstacle the tank cannot pass through. But we would want the tank to turn to a direction where there is a brick tile as it will be able to fire and destroy it. </div>

In the end, your selection for blockingLayer should be like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Inspector2.png" alt="">


Duplicate a number of tanks in the scene. Play the game now and you will realize you have Skynet Mark II! 

<img src="{{ site.baseurl }}/images/BattleCity_TankMoving3.gif" alt="">


Below is the full code for EnemyAI. [Next post]({{ site.baseurl }}{% post_url 2018-03-11-battle-city-unity-8 %}) I will show how we can do firing of shots.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyAI : Movement {
    Rigidbody2D rb2d;
    float h, v;
    [SerializeField]
    LayerMask blockingLayer;
    enum Direction { Up, Down, Left, Right };
    
    void Start()
    {
        rb2d = GetComponent<Rigidbody2D>();
        RandomDirection();
    }

public void RandomDirection()
    {
        CancelInvoke("RandomDirection");
        
        List<Direction> lottery = new List<Direction>();
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(1, 0), blockingLayer))
        {
            lottery.Add(Direction.Right);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(-1, 0), blockingLayer))
        {
            lottery.Add(Direction.Left);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, 1), blockingLayer))
        {
            lottery.Add(Direction.Up);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, -1), blockingLayer))
        {
            lottery.Add(Direction.Down);
        }
        
        Direction selection = lottery[Random.Range(0, lottery.Count)];
        if (selection == Direction.Up)
        {
            v = 1;
            h = 0;
        }
        if (selection == Direction.Down)
        {
            v = -1;
            h = 0;
        }
        if (selection == Direction.Right)
        {
            v = 0;
            h = 1;
        }
        if (selection == Direction.Left)
        {
            v = 0;
            h = -1;
        }
        Invoke("RandomDirection", Random.Range(3, 6));
    }
    private void OnCollisionEnter2D(Collision2D collision)
    {
        RandomDirection();
    }

    private void FixedUpdate()
    {
        if (v != 0 && isMoving == false) StartCoroutine(MoveVertical(v, rb2d));
        else if (h != 0 && isMoving == false) StartCoroutine(MoveHorizontal(h, rb2d));
    }
}
{% endhighlight %}
{% include serieswithintro.html %}

