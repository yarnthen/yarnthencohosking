---
layout: post
title: "Recreate Battle City in Unity Part 16 : Exploding Tank animation"
excerpt: Recreate Battle City in Unity Part 16
summary: This post is part 16 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---


This will be a short post as it is more about the animation than anything else. You will need a explosion sprite for effects.

I will be doing animation by manipulating of transform.

Start by dragging your <keyword>EnemyTank</keyword> and <keyword>PlayerTank</keyword> prefab to the hierarchy. Then add your Explosion Sprite as a child to the PlayerTank and EnemyTank. Call it <keyword>Explosion</keyword>. <keyword>Reset its transform</keyword>. 

<div class="info">You might need to adjust its scale to fit the explosion to be smaller than your tank. The animation will expand the scale to create the explosion effect</div>.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_1.png" alt="">

Then <keyword>disable the Explosion Game Object</keyword> for both the PlayerTank and EnemyTank.

Now let's do animation for the Explosion. Select your PlayerTank in the hierarchy. 

Open the Animation Window via the Unity Editor Menu(<keyword>Window -> Animation</keyword>). You should be seeing the TankCreating Animation in the Animation Window. Click on the dropdown <keyword>Create New Clip...</keyword> showing at the Animation Window. Unity will prompt for a name for the animation, call it <keyword>TankExploding</keyword>. Now you should be able to Add Property from the Animation Window. 

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_2.png" alt="">

* Add 6 properties <keyword>Explosion->Is Active</keyword>, <keyword>Explosion->Transform->Scale</keyword>, <keyword>Enemy AI->Enabled</keyword>, <keyword>Player->Enabled</keyword>, <keyword>Box Collider2D->Enabled</keyword> and <keyword>Body->Sprite Renderer->Enabled</keyword> from the Animation Window. Once added, you should be able to see something like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_3.png" alt="">

* Now click on the red circle(symbolizing record). If it is depressed, means we are in recording mode. Check the box for <keyword>Explosion: Game Object.Is Active</keyword>. Uncheck the box for <keyword>EnemyAI.Enabled, Player.Enabled, Box Collider2D.Enabled and Body Sprite Renderer.Enabled</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_4.png" alt="">

* Click on the timeline at point 0:10, you should have a white line that will move to that position. Once the white line is at point 0:10, Set value of <keyword>Scale.x and Scale.y to 1.25</keyword> and ensure the rest of the settings remains unchanged as compared to the first timeline change.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_5.png" alt="">

* Click on the timeline at point 0:20, you should have a white line that will move to that position. Once the white line is at point 0:20, Set value of <keyword>Scale.x and Scale.y to 1.5</keyword> and ensure the rest of the settings remains unchanged as compared to the first timeline change.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_6.png" alt="">

* Click on the timeline at point 0:25, you should have a white line that will move to that position. Once the white line is at point 0:25, uncheck the Explosion Game Object is Active and ensure the rest of the settings remains unchanged as compared to the first timeline change. Click on the Add Event button. A white ribbon(turns blue when selected) will appear just below the 0:25 timeline. And the inspector should come up with the title "<keyword>Animation Event</keyword>". Select from the drop-down <keyword>Death()</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_7.png" alt="">

Right-click on one of the rhombus at the point 1:00 and click Delete Key. We do not need that.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_8.png" alt="">

* Click on the red circle symbolizing record to stop recording. Now your explosion is completed. Click on Play of the Animation to start enjoying being bomberman. 

<div class="info"> You will not see the tank as its sprite renderer is set to inactive at the start of animation.</div>

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_9.gif" alt="">

The next step is to add the TankExploding Animation as a state in the Animator. If you go to the Animator Window, you will realized the TankExploding state is already added to the Base Layer. 

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_10.png" alt="">

From the Animator Window, go to the Parameters tab and create a trigger named <keyword>killed</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_11.png" alt="">

From the Animator Window, right click on the TankCreating box and select Make Transition. Drag the arrow to the TankExploding box. After the boxes are joined together by the arrow, click on the arrow and uncheck <keyword>Has Exit Time</keyword> and add condition <keyword>killed</keyword> to it. Do the same for Normal box to TankExploding Box. 

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_12.gif" alt="">

### Explosion Animation for Enemy Tank

Now we can work on the part for Enemy Tank. Create a new Animator Controller called <keyword>EnemyTank</keyword>. Select your EnemyTank and add an <keyword>Animator</keyword> Component (<keyword>Miscellaneous -> Animator</keyword>). From the Animator Component, set the Controller to point to the EnemyTank Animator Controller you created earlier.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_13.png" alt="">

Go to the Animator Window, right-click on the Animator Window Base Layer and select <keyword>Create State -> Empty</keyword>. Rename the New State as <keyword>Normal</keyword>. Drag and drop the TankExploding Animation to the Base Layer. Then go to the Parameters tab and create a trigger named <keyword>killed</keyword>. From the Animator Window, right click on the Normal box and select Make Transition. Drag the arrow to the TankExploding box. After the boxes are joined together by the arrow, click on the arrow and uncheck <keyword>Has Exit Time</keyword> and add condition <keyword>killed</keyword> to it. Do the same for Normal box to TankExploding Box.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_14.png" alt="">

### Some lines of Code required

Creating this explosion requires some little tweaks to be updated in the <keyword>Health</keyword> script.

First we need to declare 2 variables at the beginning. A Animator called <keyword>Anime</keyword> and Rigidbody2D <keyword>rb2d</keyword>.
At the Start Monobehaviour, we will get the reference to the Animator and Rigidbody2d of the tank so that we can set the trigger <keyword>killed</keyword>. to start the explosion and rigidbody to stop the tank for moving once it has exploded(if not we will see moving explosion)
{% highlight csharp%}
Animator anime;
Rigidbody2D rb2d;
//earlier codes are omitted for focus
private void Start()
{
	//earlier codes are omitted for focus
    anime = GetComponent<Animator>();
    rb2d = GetComponent<Rigidbody2D>();
}
{% endhighlight %}

Update the <keyword>TakeDamage</keyword> routine to the below. Previously, all it does was to start the Death routine. Now the Death routine will be triggered by the TankExploding animation. We will also be setting the rigidbody's velocity to zero to stop it from moving anymore.

{% highlight csharp%}
public void TakeDamage()
{
    currentHealth--;
    if (currentHealth <= 0)
    {
        rb2d.velocity = Vector2.zero;
        anime.SetTrigger("killed");
    }
}
{% endhighlight %}

Let's try out with just one enemy tank to see if the explosion animation is ok.

<img src="{{ site.baseurl }}/images/BattleCity_Explosion_13.gif" alt="">

Alrighty! Working nicely. [Next post]({{ site.baseurl }}{% post_url 2018-03-21-battle-city-unity-18 %}), we will create the Score Scene.



