---
layout: post
title: "Recreate Battle City in Unity Part 19 : Bonus Crates - Generation"
excerpt: Recreate Battle City in Unity Part 19
summary: This post is part 19 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

We are now on one of the unique features of this game which is designed to help the players: Bonus crates. 

### Information about the bonus crates
From what I gathered from Wikipedia, Battle City has 6 types of bonus crates.
1. Grenade: This will destroy all the enemy tanks in the gameplay area. The enemy tank reserve will not be affected.
2. Star: Upgrade the Player Tank Level. This can go up to Level 4. Level 2 increases the canonball speed. Level 3 allows two simultaneous canonballs to be fired. Level 4 bestowed destroy steel wall ability. The level-up carries across stages unless the tank is destroyed, which will reset the tank back to Level 1.
3. Stopwatch: Freezes all enemy tanks in the gameplay area for a period of time.
4. Shield: Make Player Tank invincible for a period.
5. Tank: Give 1 extra life to Player.
6. Shovel: Adds steel walls around the Eagle for a period of time. It also repairs any prior damage on the wall, unless the player destroys the steel wall.

* How the bonus crates gets spawned on Gameplay area is upon destroying an enemy tank which is flashing red. 
* The spawning of the flashing red tanks are by random.
* The bonus crate type spawned is also random.
* Location of the crate is random as well.

### Creating the flashing red tank
We need the flashing red tank to indicate destroying this tank will spawn a bonus crate. We will create a script on the SmallTank Prefab called <keyword>BonusTank</keyword>. We will start with the effect of the flashing red. Create a SpriteRenderer variable called <keyword>body</keyword>, a boolean variable called <keyword>isBonusTank</keyword> and 3 routines called <keyword>MakeBonusTank</keyword>, <keyword>Blink</keyword> and <keyword>IsBonusTank</keyword>. The code will be as below.
{% highlight csharp%}
using UnityEngine;

public class BonusTank : MonoBehaviour {

    SpriteRenderer body;
    bool bonusTank = false;
    public bool IsBonusTankCheck()
    {
        return bonusTank;
    }
    public void MakeBonusTank()
    {
        body = gameObject.transform.Find("Body").gameObject.GetComponent<SpriteRenderer>();
        bonusTank = true;
        InvokeRepeating("Blink", 0, 0.3f);
    }

    private void Blink()
    {
        if (body.color == Color.white) body.color = Color.Lerp(Color.white, Color.red, 0.4f);
        else body.color = Color.white;
    }
}

{% endhighlight %}

<div class="info">The MakeBonus gets the reference of the SpriteRender of Body Game Object that is a child of the Enemy Tank Game Object and set boolean bonusTank to true. Then it will invoke repetitively the Blink routine which alternates between turning the Sprite Renderer's color to red(close to red, actual red is too striking) and white. The IsBonusTank routine is to return the value of bonusTank. This is for a check if this tank is a bonus tank when it gets destroyed so it will spawn the bonus crate accordingly.</div>

Now it's time to add the random creation of the bonus tanks. We will do it in the Spawner script. All we need to do is to add the below which gives a 20% chance that a tank spawned will be a bonus tank. This will be added <keyword>immediately after</keyword> the statement of <keyword>tank = Instantiate(tanks[tankID], transform.position, transform.rotation);</keyword>
{% highlight csharp%}

if (Random.value <= 0.2)
{
    tank.GetComponent<BonusTank>().MakeBonusTank();
}
{% endhighlight %}

<div class="warning">Drag and drop the BonusTank script to the FastTank, BigTank and ArmoredTank prefabs so that they can be bonus tanks as well.</div>

Let's try it out with 20 enemy tanks and see if we will get around the 20% chance of bonus tanks. I have disabled the PlayerSpawnPoint and the Eagle to prevent moving to the next stage.

<img src="{{ site.baseurl }}/images/BattleCity_BonusCrate_1.gif" alt="">

OK. Not bad, we got a 15%(3 out of 20). Testing successful! Let'd put this frequency 0.2 elsewhere where we can change easily per level. So let's move it to LevelManager script. The updated LevelManager script will be as below with new float variables <keyword>bonusCrateRateInThisLevel</keyword> and <keyword>bonusCrateRate</keyword>.
{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelManager : MonoBehaviour
{
    [SerializeField]
    int smallTanksInThisLevel, fastTanksInThisLevel, bigTanksInThisLevel, armoredTanksInThisLevel, stageNumber;
    public static int smallTanks, fastTanks, bigTanks, armoredTanks;
    [SerializeField]
    float spawnRateInThisLevel=5, bonusCrateRateInThisLevel=0.2f;
    public static float spawnRate { get; private set; }
    public static float bonusCrateRate { get; private set; }
    private void Awake()
    {
        MasterTracker.stageNumber = stageNumber;
        smallTanks = smallTanksInThisLevel;
        fastTanks = fastTanksInThisLevel;
        bigTanks = bigTanksInThisLevel;
        armoredTanks = armoredTanksInThisLevel;
        spawnRate = spawnRateInThisLevel;
        bonusCrateRate = bonusCrateRateInThisLevel;
    }
}
{% endhighlight %}

Then we can update the Spawner script's statement for the random rate of bonus tanks generation from <keyword>if (Random.value <= 0.2)</keyword> to <keyword>if (Random.value <= LevelManager.bonusCrateRate)</keyword> and we can amend that bonus crate frequency from the Canvas Game Object inspector.

### Creating the bonus crates

Now we move on to the creation of bonus crates, and this will be handled by the <keyword>GamePlayManager</keyword> script. We will start with declaring a Game Object Array called <keyword>bonusCrates</keyword> with SerializedField so that we can add the bonus crates prefab into the GamePlayManager in the inspector for instantiation later.
{% highlight csharp%}
[SerializeField]
GameObject[] bonusCrates;
{% endhighlight %}

The key thing for the instantiation of the bonus crate is its position. We can not be instantiating the crate at locations where the player tank can't reach(Water, steel walls and outside of the gameplay area), that would look silly. To do that we will need to get references to the tilemaps so that we can determine if the location where the crate is not appearing in those tiles. Declare <keyword>Tilemap</keyword> variables <keyword>waterTilemap</keyword> and <keyword>steelTilemap</keyword> in GameObject script. We also need to add the library UnityEngine.Tilemaps to access features of tilemaps via code.

{% highlight csharp%}
using UnityEngine.Tilemaps;
public class GamePlayManager : MonoBehaviour {
Tilemap waterTilemap, steelTilemap;
//Earlier codes are omitted for focus

{% endhighlight %}

At the Start Monobehaviour, get reference to SteelTileMap and WaterTilemap using GameObject.Find.GetComponent.
{% highlight csharp%}
void Start()
{
	steelTilemap = GameObject.Find("Steel").GetComponent<Tilemap>();
	waterTilemap = GameObject.Find("Water").GetComponent<Tilemap>();
	//Earlier code are omitted for focus
}
{% endhighlight %}

We will create a routine called <keyword>InvalidBonusCratePosition</keyword> to do the check on the positions. This routine will return a boolean answer of whether the position has a steel wall or water.
{% highlight csharp%}
bool InvalidBonusCratePosition(Vector3 cratePosition)
{
    return waterTilemap.GetTile(waterTilemap.WorldToCell(cratePosition)) != null || steelTilemap.GetTile(steelTilemap.WorldToCell(cratePosition)) != null;
}
{% endhighlight %}

So we can now create the routine for generating the bonus crate. It will be called <keyword>GenerateBonusCrate</keyword>
{% highlight csharp%}
public void GenerateBonusCrate()
{
    GameObject bonusCrate = bonusCrates[Random.Range(0, bonusCrates.Length)];
    Vector3 cratePosition = new Vector3(Random.Range(-12, 12), Random.Range(-12, 13), 0);
    if (InvalidBonusCratePosition(cratePosition))
    {
        do
        {
            cratePosition = new Vector3(Random.Range(-12, 12), Random.Range(-12, 13), 0);
            if (!InvalidBonusCratePosition(cratePosition)) Instantiate(bonusCrate, cratePosition, Quaternion.identity);
        } while (InvalidBonusCratePosition(cratePosition));
    }
    else
    {
        Instantiate(bonusCrate, cratePosition, Quaternion.identity);
    }
}
{% endhighlight %}

<div class="info"> The routine starts with selecting the type of bonus crate at random. Then we select the crate position at random keeping it within the confines of the gameplay area.(-12,-12 to 12,13). We can then do a check on the position using routine InvalidBonusCratePosition. If the position is on a tile with water or steel wall, we will select another position at random until we can get one position without a steel wall or water, then we will instantiate the bonus crate at the location.</div>

### Telling GamePlayManager to create bonus crate

This is now very straightforward. All we need to do is to run the <keyword>IsBonusTankCheck</keyword> routine of BonusTank script when an enemy tank is destroyed(Death routine). If the return value is true, run the routine GenerateBonusCrate in GamePlayManager script. Full code for the updated Death routine in Health script as below.
{% highlight csharp%}
void Death()
{
    GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
    if (gameObject.CompareTag("Player"))
    {
        GPM.SpawnPlayer();
    }
    else { 
        if (gameObject.CompareTag("Small")) MasterTracker.smallTanksDestroyed++;
        else if (gameObject.CompareTag("Fast")) MasterTracker.fastTanksDestroyed++;
        else if (gameObject.CompareTag("Big")) MasterTracker.bigTanksDestroyed++;
        else if (gameObject.CompareTag("Armored")) MasterTracker.armoredTanksDestroyed++;
        if (gameObject.GetComponent<BonusTank>().IsBonusTankCheck()) GPM.GenerateBonusCrate();
    }
    Destroy(gameObject);
}
{% endhighlight %}

That's all the code we need to generate the bonus crate. [Next post]({{ site.baseurl }}{% post_url 2018-03-24-battle-city-unity-21 %}) we will talk about creating the various types of bonus crates and coding its effects starting with the additional live bonus crate.