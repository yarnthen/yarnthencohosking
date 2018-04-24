---
layout: post
title: "Recreate Battle City in Unity Part 21 : Bonus Crates - Tank Invincibility"
excerpt: Recreate Battle City in Unity Part 21
summary: This post is part 21 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

Continuing with the Bonus Crates. We will touch on the Invincibility bonus crate which is symbolized by the helmet symbol. Its effects is to grant the PlayerTank a prolonged period of invincibility. 

Start by dragging and dropping the Sprite you have for invincibility bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>Helmet</keyword>.Add a <keyword>Box Collider 2D</keyword> and assure its boundary fills the entire crate's image, then make it a trigger. Also, assign <keyword>Powerups</keyword> layer to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_1.png" alt="">

### Creating the invincibility effect

There is not much we need to do about the invincibility effect as it is just an extension of the electric aura the tank has when it first gets spawned. So all we need to do is to create a state in the Animator and add the TankCreated animation to it.

Drag and drop the PlayerTank prefab to the hierarchy and select it. Open up its Animator Window and <keyword>right-click->Create State->Empty</keyword>. Name the State as <keyword>Invincibility</keyword>.
<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_2.png" alt="">

Create a new Parameter called <keyword>Invincible</keyword>.
<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_3.gif" alt="">

* Right-click on the Entry and select <keyword>Make Transition</keyword>, drag the arrow to Invincibility. After the boxes are joined together by the arrow, click on the arrow and add condition Invincible to it. 
* Do the same for <keyword>TankCreating</keyword> box to Invincibility Box, but this time also uncheck <keyword>Has Exit Time</keyword>. 
* Do the same for <keyword>Normal</keyword> box to Invincibility Box, also uncheck <keyword>Has Exit Time</keyword>. 
* Lastly, right-click on the Invincibilty and select <keyword>Make Transition</keyword>, drag the arrow to Normal.

<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_4.gif" alt="">

After the boxes are joined together by the arrow, click on the arrow and draw down the Settings in the Inspector and set Exit Time to 10. This will mean the animation will run for 10 seconds before moving back to Normal.
<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_5.png" alt="">

Now click on the Invincibility Box again and set the Motion in the Inspector to TankCreating.

<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_6.png" alt="">

### Coding the invincibility

Now we can creating the triggering of the invincibility. Create a script called Helmet in the Helmet Game Object. Set the class to inherit from PowerUps. The full code for Helmet as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Helmet : PowerUps
{
    // Use this for initialization
    protected override void Start()
    {
        base.Start();
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        collision.gameObject.GetComponent<Animator>().SetTrigger("Invincible");
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

<div class="info">Everything is the same as the OneUp script with exception from the part from OnTriggerEnter2D. We will simply get a reference to the Animator of the collided Game Object(which can only be the PlayerTank because of the Layer Collision Matrix changes we made in Physics 2D Settings) and set trigger Invincible. Once that is done, it will destroy the game object.</div>

That's all! We can test it already. <keyword>Create a prefab of Helmet</keyword> by dragging it to the Project Window. Go to the Canvas Game Object and drag and drop the Helmet prefab to the Bonus Crates under Game Play Manager (Script), replacing the 1up prefab.
<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_7.gif" alt="">

Then press play!

<img src="{{ site.baseurl }}/images/BattleCity_BonusHelmet_8.gif" alt="">

Testing successfully! [Next stop]({{ site.baseurl }}{% post_url 2018-03-26-battle-city-unity-23 %}), will be on the Grenade Bonus Crates.
