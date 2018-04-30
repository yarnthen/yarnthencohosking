---
layout: post
title: "Battle City in Unity Part l: Single(ton) Sole Survivor"
excerpt: Battle City in Unity Part l
summary: This post is part1 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}
### Single(ton) Sole Survivor

The first thing we need to create is a GameObject which will hold information throughout the entire game play.

Start a new Unity Project selecting 2D. Start by creating an empty Game Object naming it <keyword>MasterTracker</keyword>. 

As the MasterTracker GameObject is to hold information that is required in all scenes, it should not be destroyed when moved from Scene to Scene. To do this, we will need to create a script component also called MasterTracker to store the information. For the MasterTracker to persist in every scene, we will need to create a singleton pattern in the script. 

 The rationale is to create a singleton is to ensure each scene only contains one MasterTracker GameObject(which holds the script MasterTracker to store information required to move from scene to scene), and it will be the same instance that ran on the first scene on every scene. 

 <div class="info">We will be placing a MasterTracker GameObject in every scene, this is to ensure that game will not break if we start on an alternate scene other than the first one. </div>

 So the MasterTracker script contains a <keyword>DontDestroyOnLoad</keyword> to prevent the MasterTracker GameObject from getting destroyed while moving from scene to scene. The first time the MasterTracker GameObject runs, it stores itself into a static variable (which we will call <keyword>Instance</keyword>). Then on next scene, the MasterTracker GameObject(which every scene has one) already in the scene will run and check if the static variable Instance already contains an instance of MasterTracker. If there is already one, it will destroy itself leaving the only one MasterTracker GameObject (which is loaded from the previous scene) in the scene. Code will be as below.

>To understand more of Singleton and Static, I suggest seeing [this youtube video](https://www.youtube.com/watch?v=Jzu2CjnRiwI). I believe this is the best illustration to explain the concept.

{% highlight csharp%}
static MasterTracker instance=null;

void Awake()
{
    if(instance == null)
    {
        DontDestroyOnLoad(gameObject);
        instance = this;
    }else if(instance != this)
    {
        Destroy(gameObject);
    }
}
{% endhighlight %}


Now, <keyword>reset the transform</keyword> for the MasterTracker, so it is set at position origin.(optional but good practice for housekeeping)

By now you should have two GameObjects in your Hierarchy. The <keyword>MasterTracker</keyword> and <keyword>Main Camera</keyword>. Your Hierarchy and Inspector for MasterTracker should look like this. 

<img src="{{ site.baseurl }}/images/BattleCity_UnityHierarchy1.png" alt="">

Now we need to <keyword>determine what is the information that needs to persist inter-scene</keyword>. If you did a playthrough on Battle City, 
you would notice that after every stage, it will load a scene to tabulate the score earned from the previous stage. So take a look at the below to determine what needs to be retained scene to scene.

<img src="{{ site.baseurl }}/images/BattleCity_ScoreScene.png" alt="">

From the Score scene we will need to hold information for Player Score, number of Tanks destroyed per type, the points each tank is worth, and the stage number. In additional we also need to hold information of the player lives left. So let's create <keyword>Static Public Variables</keyword> for these. 
{% highlight csharp%}
public static int smallTankDestroyed, fastTankDestroyed, bigTankDestroyed, armoredTankDestroyed;
public static int stageNumber;
public static int playerLives=3;
public static int playerScore = 0;
{% endhighlight %}


As for the points each tank is worth, we will create a private variable with <keyword>SerializedField</keyword> and public get variable each as this will allow us to change the points worth for each type of tank from Inspector and yet allow other GameObjects to access it without changing it. Example will be like below.

{% highlight csharp%}
[SerializeField]
int smallTankPoints = 100;
public int smallTankPointsWorth { get { return smallTankPoints; } }
{% endhighlight %}

So this concludes the setup of the MasterTracker which is the most important part to ensure there is continuity in the game. We might be revisiting this script to add on if we noticed something that needs to persist between scenes. We will discussing on how to create a template for Game Scene using <keyword>Tilemap</keyword> in the [next post]({{ site.baseurl }}{% post_url 2018-03-06-battle-city-unity-3 %}). The full code up till now will be as below. 

{% highlight csharp%}

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MasterTracker : MonoBehaviour {

    static MasterTracker instance=null;

    [SerializeField]
    int smallTankPoints = 100, fastTankPoints = 200, bigTankPoints= 300, armoredTankPoints=400;
    public int smallTankPointsWorth { get { return smallTankPoints; } }
    public int fastTankPointsWorth { get { return fastTankPoints; } }
    public int bigTankPointsWorth { get { return bigTankPoints; } }
    public int armoredTankPointsWorth { get { return armoredTankPoints; } }

    public static int smallTankDestroyed, fastTankDestroyed, bigTankDestroyed, armoredTankDestroyed;
    public static int stageNumber;
    public static int playerScore = 0;
  
    void Awake()
    {
        if(instance == null)
        {
            DontDestroyOnLoad(gameObject);
            instance = this;
        }else if(instance != this)
        {
            Destroy(gameObject);
        }
    }
}

{% endhighlight %}

{% include serieswithintro.html %}