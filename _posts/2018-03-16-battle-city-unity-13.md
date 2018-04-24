---
layout: post
title: "Recreate Battle City in Unity Part 12 : Level Manager"
excerpt: Recreate Battle City in Unity Part 12
summary: This post is part 12 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

I intend to cover the part of the <keyword>Game Manager</keyword> handling the <keyword>spawning of player and enemies</keyword>. Then I realized it would be tough to explain without going through the <keyword>Level Manager</keyword> first. The Level Manager is like a level editor of sorts where we use to specify the differences between each level. The differences are the <keyword>number of each type of enemy tanks in the level</keyword> and also the <keyword>Stage Number</keyword>. 

So whenever we <keyword>create a new Stage</keyword>, other than <keyword>painting the tilemaps</keyword>, all we need to do is edit the number of tanks and the Stage Number and the scene will be done. The GamePlayManager can then read from the Level Manager what is the Stage Number and display it accordingly at the Start of Stage.

### Just code and nothing else

Level Manager is only about code as it is used to store information (albeit within the Level unlike MasterTracker which is throughout the Game).
Start by creating a C# script under the <keyword>Canvas</keyword> Game Object called <keyword>LevelManager</keyword>.

* Create 4 <keyword>Public Static Int</keyword> Variable (smallTanks, fastTanks, bigTanks, armoredTanks); this is used to keep track the number of each type of tanks that can be spawned on the Stage.
* Create 5 <keyword>SerializedField Int</keyword> Variable (smallTanksInThisLevel, fastTanksInThisLevel, bigTanksInThisLevel and armoredTanksInThisLevel and stageNumberInThisLevel). This is for us to set the number of each type of tanks in each stage and stage number.

The only code execution here is 
1. The initial number of tanks passed from the SerializedField Ints to the Public Static Ints.
2. Pass the value of stageNumber to the static int stageNumber of MasterTracker
3. This will be put in the Awake Monobehaviour to ensure the values are passed in as soon as possible.

Below is all the code that is mentioned above.

{% highlight csharp%}
using UnityEngine;

public class LevelManager : MonoBehaviour {
    [SerializeField]
    int smallTanksInThisLevel, fastTanksInThisLevel, bigTanksInThisLevel, armoredTanksInThisLevel, stageNumber;
    public static int smallTanks, fastTanks, bigTanks, armoredTanks;
    private void Awake()
    {
        MasterTracker.stageNumber = stageNumber;
        smallTanks = smallTanksInThisLevel;
        fastTanks = fastTanksInThisLevel;
        bigTanks = bigTanksInThisLevel;
        armoredTanks = armoredTanksInThisLevel;
    }
}
{% endhighlight %}

With the LevelManager ready, we can move on to the [GameManager]({{ site.baseurl }}{% post_url 2018-03-17-battle-city-unity-14 %}) again to discuss how we can trigger the spawning of enemies and player.