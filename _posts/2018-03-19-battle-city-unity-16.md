---
layout: post
title: "Battle City in Unity Part 15 : Spawning Invincibility"
excerpt: Battle City in Unity Part 15
summary: This post is part 15 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}

This will be a short post as it is more about the animation than anything else. You will need an electricity aura sprite like the below for effects.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_1.png" alt="">

I will show another way of doing animation - by manipulating its transform.

Start by dragging your <keyword>PlayerTank</keyword> prefab to the hierarchy. Then add your Electricity Sprite as a child to the PlayerTank and EnemyTank. Call it <keyword>Electricity</keyword>. <keyword>Reset its transform</keyword>. 

<div class="info">You might need to adjust its scale to fit it to surround your tank.</div>. Also <keyword>set its Order in Layer to 2</keyword> so that it sits on top of all the rendered objects.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_2.png" alt="">

Then <keyword>disable the Electricity Game Object</keyword>.

Now let's do animation for the Electricity.  Go to your Project Window, create an Animation Controller and rename it as PlayerTank. Select your PlayerTank and add an <keyword>Animator</keyword> Component (<keyword>Miscellaneous -> Animator</keyword>). From the Animator Component, set the Controller to point to the PlayerTank Animator Controller you created earlier.

Open the Animation Window via the Unity Editor Menu(<keyword>Window -> Animation</keyword>). Click on "Create" at the Animation Window. Unity will prompt for a name for the animation, call it <keyword>TankCreating</keyword>. Now you should be able to Add Property from the Animation Window. 

* Add 2 properties (<keyword>Electricity->Is Active</keyword>) and (<keyword>Electricity->Transform->Rotation</keyword>) from the Animation Window.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_3.png" alt="">

Once added, you should be able to see something like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_4.png" alt="">

* Now click on the red circle(symbolizing record). If it is depressed, means we are in recording mode. Check the box for <keyword>Electricity: Game Object.Is Active</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_5.png" alt="">

* Click on the timeline at point 0:15, you should have a white line that will move to that position. Once the white line is at point 0:15, Set value of <keyword>Rotation.z to 90</keyword>and ensure <keyword>Electricity: Game Object. Is Active</keyword> is still checked.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_6.png" alt="">

* Do the same for the above at timeline 1.15.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_7.png" alt="">

* Now Click on the timeline at point 0:30, that white line should move to that position together. Once the white line is at point 0:30, set value of <keyword>Rotation.z to 180</keyword> and ensure <keyword>Electricity: Game Object. Is Active</keyword> is still checked.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_8.png" alt="">

* Do the same for the above at timeline 1.30.

* Now Click on the timeline at point 0:45, that white line should move to that position together. Once the white line is at point 0:45, set value of <keyword>Rotation.z to 270</keyword> and ensure <keyword>Electricity: Game Object. Is Active</keyword> is still checked.
Do the same at timeline 1.45.


* Now Click on the timeline at point 1:00, that white line should move to that position together. Once the white line is at point 1:00, set value of <keyword>Rotation.z to 0</keyword> and ensure Electricity: Game Object. Is Active is still checked.
* Do the same at timeline 2.00. <keyword>But uncheck Electricity: Game Object. is Active</keyword> Click on the Add Event button as indicated with the Mouse Pointer below.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_9.png" alt="">

* A white ribbon(turns blue when selected) will appear just below the 2:00 timeline. And the inspector should come up with the title "<keyword>Animation Event</keyword>". Select from the drop-down <keyword>SetHealth()</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_10.png" alt="">

* Now Click on the timeline at point 0:00, that white line should move to that position together. Once the white line is at point 0:00, click on the Add Event button as indicated with the Mouse Pointer below. A white ribbon(turns blue when selected) will appear just below the 0:00 timeline. And the inspector should come up with the title "<keyword>Animation Event</keyword>". Select from the drop-down <keyword>SetInvincible()</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_11.png" alt="">

* Click on the red circle symbolizing record to stop recording. Now your Electric Aura is completed. Click on Play of the Animation to start enjoying the feeling of invincibility.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_11.gif" alt="">

The next step is to set the TankCreating Animation as the default state as this will be the first animation that will play when the tank first appears on the Scene. Then we need to do is from the Animator add another state for the animation to transit to after electricity aura exipres. Open up the Animator Window from Unity(<keyword>Window -> Animator</keyword>). Select the Tank animator from your Project Window.

Right Click on the Animator Window Base Layer and select <keyword>Create State -> Empty</keyword>. Rename the New State as <keyword>Normal</keyword>. Your Base Layer should look like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_12.png" alt="">

Right click on the TankCreating box and select <keyword>Make Transition</keyword>. Drag the arrow to the Normal box. Select the arrow created and ensure the Inspector shows <keyword>Has Exit Time</keyword> checked

<img src="{{ site.baseurl }}/images/BattleCity_Invincibility_13.png" alt="">

### No new code required

That's all for the electricity aura effect animation. Remember to save your PlayerTank Prefab. [Next post]({{ site.baseurl }}{% post_url 2018-03-20-battle-city-unity-17 %}) we will talk about the explosion animation when a tank is destroyed.
{% include serieswithintro.html %}
s