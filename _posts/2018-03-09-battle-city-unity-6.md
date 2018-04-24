---
layout: post
title: "Recreate Battle City in Unity Part 5: Player Controller"
excerpt: Recreate Battle City in Unity Part 5
summary: This post is part 5 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

### Heir to the Movement Script

I mentioned in the previous post the Movement Script is an abstract class which needs to be inherited from to be complete. There are two scripts which will inherit from the Movement Script; one will be AI script which the computer uses to decide when and where to move, the second will be the Player Controller script which we will talk about in this post.

The script will be quite straightforward, at least for now! It will get a bit complicated later once we add in more nitty and gritty stuff. Let’s start by creating a C# script from the Project Window and name it <keyword>Player</keyword>. First, let’s set the script to inherit from Movement script by updating the class declaration to the below

{% highlight csharp%}
public class Player : Movement {
{% endhighlight %}

### Detecting Input

In the previous post, I decide on using Input.GetAxisRaw as means for input detection. The code is quite simple; it is just calling Input.GetAxisRaw and writing the value to two float variable h and v.

{% highlight csharp%}
h = Input.GetAxisRaw("Horizontal");
v = Input.GetAxisRaw("Vertical");
{% endhighlight %}

The complicated part of this is deciding where to call this code. The movement is handled via coroutine which is designed to complete at the end of a frame, so the best part to have the Input.GetAxisRaw code will be in Update as that runs at every frame. This will ensure minimal delay in getting input at the next available chance.

{% highlight csharp%}
float h, v;
void Update()
{
    h = Input.GetAxisRaw("Horizontal");
    v = Input.GetAxisRaw("Vertical");
}
{% endhighlight %}


<div class="info">You will be able to call Monobehaviour routines(e.g., Update, Start) despite inheriting from Movement Script instead of Monobehahaviour because Movement is in turn inherited from MonoBehaviour which in turns allow classes which inherit Movement to access Monobehaviour routines.</div>

### Calling the coroutine to start moving

Next part is to decide when to call the coroutine since the coroutine is to handle movement and we do not want the movement to miss any collision detection(which is physics based), so the best place to call the coroutine is at FixedUpdate which Physics calculation and update will occur after every FixedUpdate. 

Since our coroutines take in 2 parameters, 1 for the GetAxisRaw value and 1 for rigidbody2d. We will need to do a <keyword>GetComponent</keyword> at the Start Monobehaviour to get a reference to the rigidbody2d component of the Game Object we are moving. 

As we are not going to allow diagonal movement, it is necessary to prioritize horizontal or vertical input. I decided to give Horizontal the priority since evasive actions will most likely be from the right and left as we will be moving from down to up to defeat the enemies.(OK I made that up, just choose either one for priority). So just do a check for horizontal(or vertical if you preferred that) if it is of value 0 before checking on vertical(or vice-versa)

We also need to check if a movement coroutine is already in progress before starting another, so we need to check if the isMoving flag (from the Movement script) is set to false.

{% highlight csharp%}
Rigidbody2D rb2d;

void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
}

void FixedUpdate()
{
    if (h != 0 && !isMoving) StartCoroutine(MoveHorizontal(h, rb2d));
    else if (v != 0 && !isMoving) StartCoroutine(MoveVertical(v, rb2d));
}
{% endhighlight %}

### Try it out!

Let's save our code and test it. Drag and drop the Player script from the Project Window to the tank Game Object you left behind in the hierarchy. This will create a Player (script) component in the Game Object's inspector. 
1. Add a Rigidbody 2D component(<keyword>Physics 2D->Rigidbody 2D</keyword>) so that we can move it. 
2. Set the <keyword>gravity scale</keyword> in the RigidBody 2D component to 0(to prevent it from falling off!). 
3. Set the <keyword>Constraints</keyword> to check Freeze Rotation Z(to stop it from spinning when colliding with the edge of a collider).
4. Add a Box Collider 2D component(<keyword>Physics 2D->Box Collider 2D</keyword>) to the inspector to detect collisions.

Once done, your inspector for the Game Object should look like mine.

<img src="{{ site.baseurl }}/images/BattleCity_Inspector1.png" alt="">

Now let's play! Use your keyboard direction keys; it should move the tank smoothly.


### Summary & Full Code

That concludes the part on Player Controller. Short and sweet. [Next up]({{ site.baseurl }}{% post_url 2018-03-10-battle-city-unity-7 %}) is on Enemy AI. Full code of the PlayerMovement until now as below.

{% highlight csharp%}
using UnityEngine;
public class Player : Movement {
    float h, v;
    Rigidbody2D rb2d;
    void Start () {
        rb2d = GetComponent<Rigidbody2D>();
    }
    private void FixedUpdate()
    {
        if (h != 0 && !isMoving) StartCoroutine(MoveHorizontal(h, rb2d));
        else if (v != 0 && !isMoving) StartCoroutine(MoveVertical(v, rb2d));
    }
    void Update () {
        h = Input.GetAxisRaw("Horizontal");
        v = Input.GetAxisRaw("Vertical");
    }
}
{% endhighlight %}
