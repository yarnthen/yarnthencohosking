---
layout: post
title: "Recreate Battle City in Unity Part 24 : Bonus Crates - Level Up"
excerpt: Recreate Battle City in Unity Part 24
summary: This post is part 24 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

This one is probably the hardest of the bonus crates to create as it has multi-tier changes.
Let's look at what we should expect with the upgrades

* Level 2 (second tier): canonball speed increase.
* Level 3 (third tier): can fire 2 canonballs at a time.
* Level 4 (fourth tier): canonballs can destroy steel walls
There is also a tank appearance change per upgrade.

Let's go straight into creating the routine. The task will be handled by the <keyword>Player</keyword> Script as it is the difference between the Player tank and Enemy tank. First declare 3 sprite renderer variable called <keyword>level2Tank, level3Tank, level4Tank</keyword> with SerializeField. We will use this to update the tank appearance per update. Drag and drop 4 sprites of the different tank types. I reused the ones for the enemy.
{% highlight csharp%}
[SerializeField]
Sprite level2Tank, level3Tank, level4Tank;
{% endhighlight %}

Now create a public integer variable called level defaulting it to 1, this is for the tank level tracking and create a new public routine called <keyword>UpgradeTank</keyword>. Below is the skeleton code for the UpgradeTank routine.
{% highlight csharp%}
public int level=1;
public void UpgradeTank()
{
    if (level < 4)
    {
        level++;
        if (level == 2)
        {
        	//change appearance to level2Tank
            //upgrade canonball speed
        }
        else if (level == 3)
        {	
        	//change appearance to level3Tank
            //2 canonball shots at any time
        }
        else if (level == 4)
        {
        	//change appearance to level4Tank
            //canonball with destroysteel ability
        }
    }
}
{% endhighlight %}
<div class="info">There is no need to code for resetting to Level 1 when the player tank gets destroyed and respawn as the respawned tank will be the PlayerTank prefab which is with Level 1 Tank configuration</div>

We can now code for the changes due to the level up. Since all the changes are on the projectile itself, the best place to write the code will be at the <keyword>WeaponController</keyword> script. 

### Upgrade projectile speed
Create a new public routine called <keyword>UpgradeProjectileSpeed</keyword>. The code for this routine is just to update the canonball speed via its reference variable <keyword>canon</keyword>.
{% highlight csharp%}
public void UpgradeProjectileSpeed()
{
	speed = 20;
    canon.speed = speed; //initial speed is 10
}
{% endhighlight %}
That's all we need.

### 2 shots per time
This is probably the most tricky of the 3. Because we are only using a single canonball for all the shots, we will need to create a second canonball. Create a new routine called <keyword>GenerateSecondCanonBall</keyword>. We will also declare a new Projectile variable called <keyword>canon2</keyword> which is in addition to the first one named <keyword>canon</keyword> and a new GameObject variable canonBall2. Then we instantiate the new projectile at the same position as the first one and with same speed.

{% highlight csharp%}
GameObject canonBall, canonBall2, fire;
private Projectile canon, canon2;
public void GenerateSecondCanonBall()
{
    canonBall2 = Instantiate(projectile, transform.position, transform.rotation) as GameObject;
    canon2 = canonBall2.GetComponent<Projectile>();
    canon2.speed = speed;
}
{% endhighlight %}

We can now update the Fire routine in WeaponController to take in the additional projectile if it is present to have a second shot.
{% highlight csharp%}
public void Fire()
{
    if (canonBall.activeSelf == false)
    {
        //move the projectile back to Gunport
        canonBall.transform.position = transform.position;
        //set the projectile rotation to that of Gunport
        canonBall.transform.rotation = transform.rotation;
        //set the projectile to be active
        StartCoroutine(ShowFire());
        canonBall.SetActive(true);
    }
    else
    {
        if (canonBall2 != null)
        {
            if (canonBall2.activeSelf == false)
            {
                //move the projectile back to canon barrel
                canonBall2.transform.position = transform.position;
                //set the projectile rotation to canon barrel front;
                canonBall2.transform.rotation = transform.rotation;
                //set the projectile to be active
                StartCoroutine(ShowFire());
                canonBall2.SetActive(true);
            }
        }
    }
}
{% endhighlight %}

We also need to update the OnDestroy routine to take in the fact that the 2nd canonball needs to be destroyed if the tank is destroyed.
{% highlight csharp%}
private void OnDestroy()
{
    //if the tank is destroyed. Send command to destroy the projectile once inactive
    if (canonBall != null) canon.DestroyProjectile();
    if (canonBall2 != null) canon2.DestroyProjectile();
}
{% endhighlight %}

### Canonball with destroy steel wall ability

This is probably the easiest of the lot as we have done the preparation work in advance. Create a public routine called <keyword>CanonBallPowerUpgrade</keyword>. The code is simply to set the destroySteel boolean of the projectile script to true.
{% highlight csharp%}
public void CanonBallPowerUpgrade()
{
    if (canonBall != null) canon.destroySteel = true;
    if (canonBall2 != null) canon2.destroySteel = true;
}
{% endhighlight %}

### Consolidating everything

Now we can map back all the changes to the UpgradeTank routine in Player script.

{% highlight csharp%}
public void UpgradeTank()
{
    if (level < 4)
    {
        level++;
        if (level == 2)
        {
            transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level2Tank;
            wc.UpgradeProjectileSpeed();
        }
        else if (level == 3)
        {
            transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level3Tank;
            wc.GenerateSecondCanonBall();
        }
        else if (level == 4)
        {
            transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level4Tank;
            wc.CanonBallPowerUpgrade();
        }
    }
}
{% endhighlight %}

### Create the bonus crate
Now we will create the bonus crate. Start by dragging and dropping the Sprite you have for Level up bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>LevelUp</keyword>.Add a <keyword>Box Collider 2D</keyword> and assure its boundary fills the entire crate's image, then make it a trigger. Also, assign <keyword>Powerups</keyword> layer to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_LevelUp_1.png" alt="">

Now we will create the code for the LevelUp crate. Add a new script called <keyword>LevelUp</keyword> to the LevelUp Game Object. Set the class to inherit from PowerUps. The full code for LevelUp as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelUp : PowerUps
{
    protected override void Start()
    {
        base.Start();
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        collision.gameObject.GetComponent<Player>().UpgradeTank();
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

Now we are ready to try out! I updated the map with Steel Walls to test all the upgrades.

<img src="{{ site.baseurl }}/images/BattleCity_LevelUp_2.gif" alt="">

### Level up tank to next stage

One of the important features for the Level Up crate is that the level upgrade gets carried over to next stage. For the information to get carried over to the next stage we will need something that does not get destroyed when moving to next scene, so we will need <keyword>MasterTracker</keyword>. Declare a new public static integer called <keyword>playerLevel</keyword> defaulting its value to 1.

{% highlight csharp%}
public static int playerLevel = 1;
{% endhighlight %}

Then we need to pass the information of the Tank Level from the stage to MasterTracker before it completes, the place to do this will be the <keyword>LevelCompleted</keyword> routine in <keyword>GamePlayManager</keyword>. Full code for the routine as below.

{% highlight csharp%}
void LevelCompleted()
{
    tankReserveEmpty = false;
    Player player = GameObject.FindGameObjectWithTag("Player").GetComponent<Player>();
    MasterTracker.playerLevel = player.level;
    SceneManager.LoadScene("Score");
}
{% endhighlight %}

Then we can add the code of upgraded tank. This will be done at the <keyword>Start</keyword> Mononehaviour of the <keyword>Player</keyword> script. This would mean the Player Script will check MasterTracker for the tank level before creating the Player Tank at the appropriate level.
{% highlight csharp%}
void Start () {
    wc = GetComponentInChildren<WeaponController>();
    rb2d = GetComponent<Rigidbody2D>();
    if (MasterTracker.playerLevel > 1)
    {
        transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level2Tank;
        wc.level = 2;
        if (MasterTracker.playerLevel > 2)
        {
            transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level3Tank;
            wc.level = 3;
            if (MasterTracker.playerLevel > 3)
            {
                transform.Find("Body").gameObject.GetComponent<SpriteRenderer>().sprite = level4Tank;
                wc.level = 4;
            }
        }
    }
}
{% endhighlight %}
<div class="info">The code above only changes the Sprite of the tank to its appropriate level. We will update a new public integer variable in WeaponController Script to complete adding the necessary powerups due to the level up.</div>

Now we go to the WeaponController to add in the code for the powerups. Same as the Player script, we will be adding in the Start Monobehaviour. Also remember to create the public integer variable <keyword>level</keyword>. The full code as below.
{% highlight csharp%}
public int level=1;
void Start () {
    canonBall = Instantiate(projectile, transform.position, transform.rotation) as GameObject;
    canon = canonBall.GetComponent<Projectile>();
    canon.speed = speed;
    fire = transform.GetChild(0).gameObject;
    if (level > 1) UpgradeProjectileSpeed();
    if (level > 2) GenerateSecondCanonBall();
    if (level > 3) CanonBallPowerUpgrade();
}
{% endhighlight %}
<div class="info">If you followed carefully, you would have realized we could just have called the powerup routines(UpgradeProjectileSpeed, GenerateSecondCanonBall, CanonBallPowerUpgrade) from the Player script as we already have a reference to the WeaponController there. I tried that, and it did not work because the powerup routines would have run before the Start Monobehaviour of WeaponController can run. This will result in the famous "object reference not set to an instance of an object" error.</div>

The tank's level will also be reset if it gets destroyed and respawns. So we need to update the Death routine in Health script so that it will reset the MasterTracker's playerLevel value to 1.The updated code for Death routine as below.
{% highlight csharp%}
void Death()
{
    GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
    if (gameObject.CompareTag("Player"))
    {
        MasterTracker.playerLevel = 1;
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

Now 1 last test to see if the level gets carried over to the next stage. We will put 3 LevelUp crates in the scene and leave only 1 enemy so we can destroy it quickly and see if the level up gets moved over.

<img src="{{ site.baseurl }}/images/BattleCity_LevelUp_3.gif" alt="">

All cleared! Let's move to the [last bonus crate- Spade]({{ site.baseurl }}{% post_url 2018-03-29-battle-city-unity-26 %}).