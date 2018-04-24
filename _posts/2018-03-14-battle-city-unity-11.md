---
layout: post
title: "Recreate Battle City in Unity Part 10: The Spawn"
excerpt: Recreate Battle City in Unity Part 10
summary: This post is part 10 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

### How does a spawn look like?

Let's start by seeing a Spawn in action(Screenshot courtesy of Youtube)<br>
<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_1.gif" alt="">

So all we need is a Star. Let's get to it. Drag and drop a Star Sprite into the Hierarchy, rename the Game Object created as <keyword>Star</keyword> and reset its transform. Create an empty Game Object calling it <keyword>SpawnPoint</keyword>, make Star its child and <keyword>reset the child's transform</keyword>.
<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_2.png" alt="">

### Some Coding

Add a new C# script to the SpawnPoint GameObject calling it <keyword>Spawner</keyword>. Declare <keyword>GameObject array</keyword> variable named <keyword>tanks</keyword>. This is to store references to the prefabs of all types of tanks. Declare a GameObject called <keyword>tank</keyword>. This is to store reference to the tank instantiated later. Declare 4 GameObject called <keyword>smallTank, fastTank, bigTank, armoredTank</keyword> with SerializedField to store the prefabs to be used by tanks array. Declare a <keyword>boolean</keyword> variable named <keyword>isPlayer</keyword> with SerializedField. This is to differentiate if the SpawnPoint is for player or enemy.
{% highlight csharp%}
GameObject[] tanks;
GameObject tank;
[SerializeField]
bool isPlayer;
[SerializeField]
GameObject smallTank, fastTank, bigTank, armoredTank;
{% endhighlight %}
At the Start Monobehaviour routine of Spawner, add the below code to store the references to the prefabs of the tanks to tanks array (4 for enemy and smallTank Game Object only for the player)

{% highlight csharp%}
void Start()
{
    if (isPlayer)
    {
        tanks = new GameObject[1] { smallTank };
    }
    else
    {
        tanks = new GameObject[4] { smallTank, fastTank, bigTank, armoredTank };
    }
}
{% endhighlight %}

Create a public routine called <keyword>StartSpawning</keyword>. Below is the full code which is to instantiate one of the tank prefabs randomly at the location of the SpawnPoint.
{% highlight csharp%}
public void StartSpawning()
{
    tank = Instantiate(tanks[Random.Range(0,tanks.Length)], transform.position, transform.rotation);
}
{% endhighlight %}

Create a public routine called <keyword>SpawnNewTank</keyword>. Below is the full code which is to set the tank instantiated to be active. The instantiated tank is initially disabled at the start of the animation and at the end of the animation, we will run the SpawnNewTank routine to make it active.
{% highlight csharp%}
public void SpawnNewTank()
{
    if (tank != null) tank.SetActive(true);
}
{% endhighlight %}


### Twinkle Twinkle Little Star

Now let's do some animation for the Star. We are using animation as a means of a timer(it takes about 1 second from the Star twinkling to spawning a tank). Go to your Project Window, create an <keyword>Animation Controller</keyword> and rename it as SpawnPoint. Move the Animation Controller to your preferred folder for organization. Select the SpawnPoint Game Object from the hierarchy and add an <keyword>Animator</keyword> Component (<keyword>Miscellaneous -> Animator</keyword>). From the Animator Component, set the Controller to point to the SpawnPoint Animator Controller you created earlier.
<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_3.gif" alt="">

Open the Animation Window via the Unity Editor Menu(<keyword>Window -> Animation</keyword>). Ensure your SpawnPoint Game Object is selected in the Hierarchy, click on "Create" at the Animation Window. Unity will prompt for a name for the animation, call it <keyword>StarTwinkle</keyword>. Now you should be able to Add Property from the Animation Window. Add a property (<keyword>Transform->Scale</keyword>) from the Animation Window.
<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_3.png" alt="">

Once added, expand the drop-down for <keyword>SpawnPoint:Scale</keyword>, you should be able to see something like the below.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_4.png" alt="">

Now click on the red circle(symbolizing record). If it is depressed, means we are in recording mode. Click on the timeline at point 0:15, you should have a white line that will move to that position. Once the white line is at point 0:15, Set value of <keyword>Scale.x and Scale.y to 1.5</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_5.png" alt="">

Do the same for the above at timeline 0.45.

Now Click on the timeline at point 0:30, that white line should move to that position together. Once the white line is at point 0:30, Set value of Scale.x and Scale.y to 1.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_6.png" alt="">

Do the same for the above at timeline 1.00.

Now Click on the timeline at point 1:10, that white line should move to that position together. Once the white line is at point 1:10, set the value of Scale.x and Scale.y to 0. Click on the Add Event button as indicated with the Mouse Pointer below.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_7.png" alt="">

A white ribbon(turns blue when selected) will appear just below the 1.10 timeline. And the inspector should come up with the title "<keyword>Animation Event</keyword>". Select from the drop-down <keyword>SpawnNewTank()</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_8.png" alt="">

Now Click back to the timeline at point 0:00, that white line should move to that position together. Once the white line is at point 0:00Click on the Add Event button as indicated with the Mouse Pointer below. A white ribbon(turns blue when selected) will appear just below the 0.00 timeline. And the inspector should come up with the title "<keyword>Animation Event</keyword>". Select from the drop-down <keyword>StartSpawning()</keyword>.
<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_9.png" alt="">

Click on the red circle symbolizing record to stop recording. Now your Star Twinkle animation is completed. Click on Play of the Animation to start singing Twinkle Twinkle Little Stars along with it.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_9.gif" alt="">

### Spawning the Enemies and Player

We are almost done. Now duplicate the SpawnPoint naming the duplicate <keyword>PlayerSpawnPoint</keyword>. Drag and drop the <keyword>PlayerTank</keyword> prefab to the smallTank variable <keyword>only</keyword> of PlayerSpawnPoint's inspector and check the box for <keyword>isPlayer</keyword>. Drag and drop the <keyword>EnemyTank</keyword> prefab to smallTank, fastTank, bigTank and armoredTank variable of SpawnPoint's inspector. 

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_10.gif" alt="">

Now let's place the SpawnPoint and PlayerSpawnPoint at different locations to see the spawn happening!

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_11.gif" alt="">

Ok. Not that good a demonstration. We will need to stop the animation from running, again and again, to prevent it spamming(I mean spawning).

Select the StarTwinkle animation and uncheck the <keyword>Loop Time</keyword> from its inspector.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_12.png" alt="">

Let's try again now!

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_13.gif" alt="">

### Adding the finishing touches

Now to complete the spawning effects. Select both the SpawnPoint and PlayerSpawnPoint. Set both its Transform.Scale.x and Transform.Scale.y to 0. This is to make the Star disappear from view. The Star does not appear by default but only whenever there is a need to spawn a tank. So it is best to set it to invisible.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_14.gif" alt="">

The next step is a means to trigger the spawning. As of now, we can only trigger it once which is by default. What we need to do is from the Animator add another state for the animation to transit to after spawning while waiting for another call to do the spawning again. Open up the Animator Window from Unity(<keyword>Window -> Animator</keyword>). Select the SpawnPoint animator from your Project Window.

Right Click on the Animator Window Base Layer and select <keyword>Create State -> Empty</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_15.gif" alt="">

Rename the New State as <keyword>Normal</keyword> then right click <keyword>Set as Layer Default State</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_16.gif" alt="">

From the Animator Window, go to the Parameters tab and create a trigger named <keyword>Spawning</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_17.gif" alt="">

From the Animator Window, right click on the Entry box and select <keyword>Make Transition</keyword>. Drag the arrow to the StarTwinkle box. After the boxes are joined together by the arrow, click on the arrow and add condition Spawning to it. Do the same for Normal box to StarTwinkle Box.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_18.gif" alt="">

Add a Transition back from StarTwinkleBox to Normal.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_19.gif" alt="">

Now let's test the trigger for spawning.

<img src="{{ site.baseurl }}/images/BattleCity_TheSpawn_20.gif" alt="">

Perfect! So that is how we can do spawning. <keyword>Prefab both the SpawnPoint and PlayerSpawnPoint</keyword>. All that's left is for something to trigger the spawning. And that will be handled by the [GameManager]({{ site.baseurl }}{% post_url 2018-03-15-battle-city-unity-12 %}).


