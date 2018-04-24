---
layout: post
title: "Recreate Battle City in Unity Part 20 : Bonus Crates - Tank Extra Live"
excerpt: Recreate Battle City in Unity Part 20
summary: This post is part 20 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

Let's start with the easiest of the Bonus Crates - Tank bonus crate which grants 1 extra live to the player.

Start by dragging and dropping the Sprite you have for Tank bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>1up</keyword>.Add a <keyword>Box Collider 2D</keyword> and ensure its boundary fills the entire crate's image, then make it a trigger. Also, create and assign a new Layer called <keyword>Powerups</keyword> to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_BonusLive_1.png" alt="">

Go to the <keyword>Edit->Project Settings->Physics 2D</keyword> and update the collision such that only PlayerTank can collide with PowerUps layer.

<img src="{{ site.baseurl }}/images/BattleCity_BonusLive_2.png" alt="">

### Coding the 1 up

All the Crates appearing on the screen has a blinking effect on them so we will create an abstract class script to hold the code for the blinking effect, then the script to code the individual effects of the different crates will inherit from it. The abstract class script will be called <keyword>PowerUps</keyword>. The code as below.
{% highlight csharp%}
using UnityEngine;

public abstract class PowerUps : MonoBehaviour {
    private SpriteRenderer sprite;
    protected virtual void Start () {
        sprite = GetComponent<SpriteRenderer>();
        InvokeRepeating("Blink", 0, 0.1f);
    }
    void Blink()
    {
        sprite.enabled = !sprite.enabled;
    }
}
{% endhighlight %}

<div class="info"> The code gets the SpriteRenderer component of the bonus crate and repetitively turn on and off the sprite renderer. The protected virtual at the Start Monobehaviour is to allow classes inheriting from PowerUps to call and modify the Start MonoBehaviour codes written in the abstract class(running the GetComponent from PowerUps won't work as the script attached to the bonus crate will be the script inheriting from PowerUps)</div>

Now we start creating the effects of the bonus live. Create a script called OneUp in the 1up Game Object. Set the class to inherit from PowerUps. The full code for OneUp as below.
{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class OneUp : PowerUps
{
    protected override void Start()
    {
        base.Start();
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        MasterTracker.playerLives++;
        GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
        GPM.UpdatePlayerLives();
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

<div class="info">The code starts by triggering <keyword>base.Start</keyword> which gets the code from its parent class PowerUps Start Monobehaviour to run. Then in the <keyword>OnTriggerEnter2D</keyword>, add one life to <keyword>playerLives</keyword> which is a static variable in <keyword>MasterTracker</keyword>. Then run the routine <keyword>UpdatePlayerLives</keyword> which will update the Battle Status Board of the number of lives the player now has. Once that is done, it will destroy the game object.</div>

That's all! We can test it already. <keyword>Create a prefab of 1up</keyword> by dragging it to the Project Window. Go to the Canvas Game Object and drag and drop the 1up prefab to the Bonus Crates under Game Play Manager (Script). We can also test the crate to not appear at silly positions effect by overflowing the gameplay area with Water tiles. We will also set the number of enemy tanks to spawn to 5 with a 100% chance of it being a bonus tank.

<img src="{{ site.baseurl }}/images/BattleCity_BonusLive_3.gif" alt="">

Testing successfully! We see that the crates only get spawned in areas where we can reach and also the effects(plus 1 life) is working! You can prefab the 1up now. Next post we will talk about the invincibility crate.
