---
layout: post
title: "Battle City in Unity Part 22 : Bonus Crates - Grenade"
excerpt: Battle City in Unity Part 22
summary: This post is part 22 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}
Next up on the Bonus Crates. We will touch on the Grenade bonus crate which is symbolized by the grenade symbol. Its effect is to destroy all enemy tanks in the gameplay area instantly, but the enemy tank reserve will not be affected.

Start by dragging and dropping the Sprite you have for grenade bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>Grenade</keyword>.Add a <keyword>Box Collider 2D</keyword> and assure its boundary fills the entire crate's image, then make it a trigger. Also, assign <keyword>Powerups</keyword> layer to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top 
of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_Grenade_1.png" alt="">


### Creating the grenade effect

The key to creating the grenade effect is to be able to find all the enemy tanks on the scene. But if you followed faithfully through all these tutorials, you would have realized all the enemy tanks clones are all around the hierarchy, and it looks very messy. So we will need a means to keep them neatly using an empty Game Object as a parent to "look after them". Create an empty Game Object in the hierarchy calling it <keyword>EnemyHolder</keyword> making it child of <keyword>Stage Essential</keyword> Game Object and reset its transform.

<img src="{{ site.baseurl }}/images/BattleCity_Grenade_2.png" alt="">

### Code modification

Now to move all the spawned enemies we need to get a reference to <keyword>Transform of EnemyHolder</keyword> Game Object in <keyword>Spawner</keyword> script. We do that by first declaring a <keyword>Transform</keyword> variable called <keyword>enemyHolder</keyword>, then get the reference in the Start Monobehaviour using <keyword>GameObject.Find</keyword>.

{% highlight csharp%}
Transform enemyHolder;
void Start()
{
    enemyHolder = GameObject.Find("EnemyHolder").transform;
    //Earlier codes are omitted for focus
}
{% endhighlight %}

Then all that's left is to make a newly instantiated enemy tank as a child to the enemyHolder in the StartSpawning routine in Spawner script. The updated code for StartSpawning routine will be as below.
{% highlight csharp%}
public void StartSpawning()
{
    if (!isPlayer)
    {
        List<int> tankToSpawn = new List<int>();
        tankToSpawn.Clear();
        if (LevelManager.smallTanks > 0) tankToSpawn.Add((int)tankType.smallTank);
        if (LevelManager.fastTanks > 0) tankToSpawn.Add((int)tankType.fastTank);
        if (LevelManager.bigTanks > 0) tankToSpawn.Add((int)tankType.bigTank);
        if (LevelManager.armoredTanks > 0) tankToSpawn.Add((int)tankType.armoredTank);
        int tankID = tankToSpawn[Random.Range(0, tankToSpawn.Count)];
        tank = Instantiate(tanks[tankID], transform.position, transform.rotation);
        tank.transform.SetParent(enemyHolder); //set parent to enemyholder
        if (Random.value <= LevelManager.bonusCrateRate)
        {
            tank.GetComponent<BonusTank>().MakeBonusTank();
        }
        if (tankID == (int)tankType.smallTank) LevelManager.smallTanks--;
        else if (tankID == (int)tankType.fastTank) LevelManager.fastTanks--;
        else if (tankID == (int)tankType.bigTank) LevelManager.bigTanks--;
        else if (tankID == (int)tankType.armoredTank) LevelManager.armoredTanks--;
        GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
        GPM.RemoveTankReserve();
    }
    else
    {
        tank = Instantiate(tanks[0], transform.position, transform.rotation);
    }
}
{% endhighlight %}

Let's test to see if the Enemy Tanks are now "behaving properly" under their parent EnemyHolder.

<img src="{{ site.baseurl }}/images/BattleCity_Grenade_3.gif" alt="">

Now they are obedient children, and the hierarchy looks tidier too. Now we can get hold of all the enemy tanks in the scene easily by asking its "parent". Let's start to code on the Grenade. Add a new script called <keyword>Grenade</keyword> to the Grenade Game Object. Set the class to inherit from PowerUps. The full code for Grenade as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Grenade : PowerUps
{
    GameObject[] enemies;
    GameObject enemyHolder;
    // Use this for initialization
    protected override void Start()
    {
        base.Start();
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        enemyHolder = GameObject.Find("EnemyHolder");
        Health[] enemiesHealthScript = enemyHolder.GetComponentsInChildren<Health>(true);
        foreach (Health enemyHealthScript in enemiesHealthScript)
        {
            enemyHealthScript.TakeDamage();
        }
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

<div class="info">Everything is the same as the OneUp script with exception from the part from OnTriggerEnter2D. We will get a reference to EnemyHolder GameObject and get the Health Script component references to all the children of EnemyHolder(which are all the enemy tanks) so that we can activate the TakeDamage routine in their health script to destroy them.<br>
The boolean parameter value of true is to get the references of those inactive Game Objects as well(the tanks are disabled at the start of spawning animation). Once that is done, it will destroy the game object.</div>

Let's also update the <keyword>TakeDamage</keyword> script to take in the amount of damage it should be given. We will set that as an optional parameter so that we will not need to change any earlier usage of the routine. We will also need an optional boolean parameter(it is <keyword>divineIntervention</keyword>) to inform <keyword>MasterTracker</keyword> to not add tanks which are destroyed by the grenade in the statistics. This is done by updating the Death routine in Health Script. Updated routine for TakeDamage and Death as below.
{% highlight csharp%}
bool divineIntervention;
public void TakeDamage(int damage=1, bool destroyedByPowerUp=false)
{
    divineIntervention = destroyedByPowerUp;
    currentHealth-=damage;
    if (currentHealth <= 0)
    {
        rb2d.velocity = Vector2.zero;
        anime.SetTrigger("killed");
    }
}
void Death()
{
    GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
    if (gameObject.CompareTag("Player"))
    {
        GPM.SpawnPlayer();
    }
    else {
        if (!divineIntervention)
        {
            if (gameObject.CompareTag("Small")) MasterTracker.smallTanksDestroyed++;
            else if (gameObject.CompareTag("Fast")) MasterTracker.fastTanksDestroyed++;
            else if (gameObject.CompareTag("Big")) MasterTracker.bigTanksDestroyed++;
            else if (gameObject.CompareTag("Armored")) MasterTracker.armoredTanksDestroyed++;
        }
        if (this.gameObject.GetComponent<BonusTank>().IsBonusTankCheck()) GPM.GenerateBonusCrate();
    }
    Destroy(gameObject);
}
{% endhighlight %}

Then we can update the OnTriggerEnter2D of Grenade script to add in the parameter of TakeDamage to a large number so that it can destroy even those newly spawned enemy tanks(initial invincibility grants them 1000 health points) and also setting the flag of destroyedByPowerUp to true.

{% highlight csharp%}
private void OnTriggerEnter2D(Collider2D collision)
{
    enemyHolder = GameObject.Find("EnemyHolder");
    Health[] enemiesHealthScript = enemyHolder.GetComponentsInChildren<Health>(true);
    foreach (Health enemyHealthScript in enemiesHealthScript)
    {
        enemyHealthScript.TakeDamage(10000,true);
    }
    Destroy(this.gameObject);
}
{% endhighlight %}

Now we can test. I will delibrately destroy just 1 tank while leaving the rest to be destroyed by the grenade and see if the Score Scene takes in the accurate information.

<img src="{{ site.baseurl }}/images/BattleCity_Grenade_4.gif" alt="">

Working as expected. You can prefab the Grenade now. Let's move to the harder Bonus Crate effects starting with [Stopwatch]({{ site.baseurl }}{% post_url 2018-03-27-battle-city-unity-24 %}).
{% include serieswithintro.html %}
