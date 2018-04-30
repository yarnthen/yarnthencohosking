---
layout: post
title: "Battle City in Unity Part 4: Tank Movement"
excerpt: Battle City in Unity Part 4
summary: This post is part 4 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}

### Prep Works

Now I want you to delete the rest of the tanks you have created leaving just one. You know that it is straightforward to create the rest of the tanks so we can do that again later. We only need one tank right now as we will be making a lot of modification on this Game Object and the same modification will be made on all the types of tank. So might as well we do all the modification on one then duplicate them at the end.

Next up, we will taking care of the movement of the tanks. They are main moving objects in this game. The enemy tanks and player tank share similar movement except that one is controlled by the computer while the other is controlled by the player itself. What I am going to do is to create a class to hold all the movement essentials which shared by the Enemy Tanks and Player Tanks. With this, we will make this class an <keyword>abstract class</keyword> which means this is an incomplete class that cannot exist on its own without another non-abstract class inheriting from it.

To create such a class, we will create a C# script from the Project Window and name it Movement. Open the Movement script in Monodevelop or Visual Studio whichever you preferred. Insert the word abstract to the class declaration line in the Movement script so it will read as the below.

{% highlight csharp%}
public abstract class Movement : MonoBehaviour {
{% endhighlight %}

Go to Youtube to observe the movement of the tanks done in the Battle City Game. You will realize: 
1. The tanks will turn to face the direction it is moving before moving forward.
2. The tanks will only move in up, down, left, right directions(no diagonals).
3. The movement of the tanks are perfect in a sense it will definitely turn at the exact per unit(e.g., 1,1 or 1,2) point. The perfect movement, in turn, will prevent scenarios like getting trapped as moving to a sub-unit point (e.g., 1,1.23  or 2.3,1.57) will mean unnecessary collisions with the tiles which are lined up precisely in per unit point. These collisions in turn affect the gameplay experience.
4. Movement is smooth and not jerky.

### Tank Movement Plans

There needs to be some input for the script to determine which direction to move.  Since no input will be expected by the enemy as that is part of AI, we will look at how we can get user input. We will get input from the user is via the input manager using the command <keyword>Input.GetAxisRaw</keyword> with string parameter of <keyword>Horizontal</keyword> and <keyword>Vertical</keyword>. These are defaulted to return whole number value based on key depressed on a controller D-pad, Up Down Left Right and ASWD of the keyboard. So if we input the Left Arrow on the keyboard, Input.GetAxisRaw("Horizontal") will return -1 and Up Arrow will have <keyword>Input.GetAxisRaw("Vertical")</keyword> to return 1. 

We will not be allowing diagonal movement, so we will split the routine into Horizontal Movement and Vertical Movement to have better control over which movement has priority. I created two routines called <keyword>MoveHorizontal</keyword> and <keyword>MoveVertical</keyword> taking in two parameters of <keyword>movementVertical</keyword>/<keyword>movementHorizontal</keyword>(which will be +1 or -1 to determine up down right left based on Input.GetAxisRaw) and also <keyword>rigidbody2d</keyword> for doing <keyword>MovePosition</keyword>. These two routines will be <keyword>coroutine</keyword> so that I will be able to do smooth movement using yield as a means to allow frame by frame execution as opposed to using Update/FixedUpdate/LateUpdate. I usually use these monobehaviours for detection purposes only. 

The coroutines will be with <keyword>protected</keyword> permission so that the class(which will be used to detect the Input.GetAxis for the player or used for AI for the enemy) inherited from it can call the coroutines.

{% highlight csharp%}
protected IEnumerator MoveHorizontal(float movementHorizontal, Rigidbody2D rb2d)
{
    //code for moving left and right
}
protected IEnumerator MoveVertical(float movementVertical, Rigidbody2D rb2d)
{
    //code for moving up and down   
}
{% endhighlight %}


### Tank Turning

The first part we need to handle is the rotation(turning of direction). The code is pretty much straightforward, we just need to set the rotation using <keyword>Quaternion.Euler</keyword>. Below are the values:
<div class="info">
<b>Turning to face right:</b>  Quaterion.Euler(0,0,270) or Quaterion.Euler(0,0,-90)<br>
<b>Turning to face left:</b> Quaterion.Euler(0,0,90) or Quaterion.Euler(0,0,-270)<br>
<b>Turning to face top:</b> Quaterion.Euler(0,0,0) or Quaterion.Euler(0,0,360)<br>
<b>Turning to face down:</b> Quaterion.Euler(0,0,180) or Quaterion.Euler(0,0,-180)
</div>
{% highlight csharp%}
Quaternion rotation = Quaternion.Euler(0,0,-movementHorizontal * 90f);
transform.rotation = rotation;
{% endhighlight %}

<div class="info">The above code is for left and right rotation(turning). Primarily by multiplying the negative value of movementHorizontal(which is 1 or -1) with 90 will ensure the Quaterion.Euler value at the Z axis to point to the correct direction for horizontal facing.</div>

### Tank Moving Failsafe

There are 2 <keyword>failsafe</keyword> that I add in to ensure that tanks getting trapped scenario is kept to the minimum. The first will be to make sure the tank to be at an exact unit position (e.g., 1,1 or 2,3) before performing a movement. To do this, we will set the transform.position of the tank to the nearest whole number position using <keyword>Mathf.Round</keyword> at the beginning of the execution of the movement coroutine.

{% highlight csharp%}
transform.position = new Vector2(Mathf.Round(transform.position.x), Mathf.Round(transform.position.y));
{% endhighlight %} 

The second one is a boolean(i call it <keyword>isMoving</keyword>) to detect if the tank is performing a movement. So when we start a movement, it will set isMoving to true and will only set it to false when the coroutine completes all its instructions. So from the outside, we will do a check to see if isMoving is false before triggering another movement. This failsafe ensures that the tank will always end up at an exact unit point when stopping as each execution of the movement coroutine is designed to move only 1 unit space in the required direction. If we were to allow overlapping execution of coroutines, it would cause extreme jerky movement due to the first failsafe I put in. 

The isMoving flag will also be of protected permission so that the class inheriting it can determine if the execution of the coroutine is completed below triggering the next movement.

{% highlight csharp%}
protected bool isMoving = false;
protected IEnumerator MoveHorizontal(float movementHorizontal, Rigidbody2D rb2d)
{
    isMoving = true;
    //code for moving left and right
    isMoving = false;
}
protected IEnumerator MoveVertical(float movementVertical, Rigidbody2D rb2d)
{
    isMoving = true;
    //code for moving up and down
    isMoving = false;
}
{% endhighlight %}

### Tank Movement

Although each execution of the movement coroutine is guaranteed(at least in theory) to move 1 unit space, we cannot be moving it abruptly to its destination. Instead, it should be gradual to ensure there is continuity when moving to the next execution of the movement coroutine if not we will end up with a visual faux pax.

To do this, we will be doing a <keyword>Rigidbody2d.MovePosition</keyword> gradually to its destination. The rate taken to reach its destination is determined by a <keyword>speed</keyword> variable which we will expose publicly to allow flexibility to set the speed for each tank type from the inspector.

{% highlight csharp%}
public int speed = 5;
Vector2 movement, endPos;

while (movementProgress < Mathf.Abs(movementVertical))
{
    movementProgress += speed * Time.deltaTime;
    movementProgress = Mathf.Clamp(movementProgress, 0f, 1f);

    movement = new Vector2(0f, speed * Time.deltaTime * movementVertical);
    endPos = rb2d.position + movement;

    if (movementProgress == 1) endPos = new Vector2(endPos.x, Mathf.Round(endPos.y));
    rb2d.MovePosition(endPos);
    yield return new WaitForFixedUpdate();
}


{% endhighlight %}

<div class="info">The above code is for the up-down movement. <keyword>movementProgress</keyword> checks the progress towards the destination and is clamped to 1 so that it will not overshoot its target position. The if statement is another failsafe to ensure the last gradual movement in the coroutine move to the exact location of the destination by using Mathf.Round. The progressing is via the WaitForFixedUpdate which means the increment in distance is done every FixedUpdate.</div>

### Summary and Full code

This marks the end of tank movement creation. [Next post]({{ site.baseurl }}{% post_url 2018-03-09-battle-city-unity-6 %}) we will talk about how to allow the player to control the tank.


{% highlight csharp%}
using System.Collections;
using UnityEngine;

public abstract class Movement : MonoBehaviour
{
    public int speed = 5;
    protected bool isMoving = false;

    protected IEnumerator MoveHorizontal(float movementHorizontal, Rigidbody2D rb2d)
    {
        isMoving = true;

        transform.position = new Vector2(Mathf.Round(transform.position.x), Mathf.Round(transform.position.y));

        Quaternion rotation = Quaternion.Euler(0, 0, -movementHorizontal * 90f);
        transform.rotation = rotation;
        
        float movementProgress = 0f;
        Vector2 movement, endPos;

        while (movementProgress < Mathf.Abs(movementHorizontal))
        {
            movementProgress += speed * Time.deltaTime;
            movementProgress = Mathf.Clamp(movementProgress, 0f, 1f);
            movement = new Vector2(speed * Time.deltaTime * movementHorizontal, 0f);
            endPos = rb2d.position + movement;

            if (movementProgress == 1) endPos = new Vector2(Mathf.Round(endPos.x), endPos.y);
            rb2d.MovePosition(endPos);
            
            yield return new WaitForFixedUpdate();
        }

        isMoving = false;
    }

    protected IEnumerator MoveVertical(float movementVertical, Rigidbody2D rb2d)
    {
        isMoving = true;

        transform.position = new Vector2(Mathf.Round(transform.position.x), Mathf.Round(transform.position.y));

        Quaternion rotation;

        if (movementVertical < 0)
        {
            rotation = Quaternion.Euler(0, 0, movementVertical * 180f);
        }
        else
        {
            rotation = Quaternion.Euler(0, 0, 0);
        }
        transform.rotation = rotation;

        float movementProgress = 0f;       
        Vector2 endPos, movement;
 
        while (movementProgress < Mathf.Abs(movementVertical))
        {
            
            movementProgress += speed * Time.deltaTime;
            movementProgress = Mathf.Clamp(movementProgress, 0f, 1f);

            movement = new Vector2(0f, speed * Time.deltaTime * movementVertical);
            endPos = rb2d.position + movement;
            
            if (movementProgress == 1) endPos = new Vector2(endPos.x, Mathf.Round(endPos.y));
            rb2d.MovePosition(endPos);
            yield return new WaitForFixedUpdate();
            
        }

        isMoving = false;

    }
}
{% endhighlight %}
{% include serieswithintro.html %}

