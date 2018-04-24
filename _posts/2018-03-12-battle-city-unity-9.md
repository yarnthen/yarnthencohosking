---
layout: post
title: "Recreate Battle City in Unity Part 8: Hurting the tanks"
excerpt: Recreate Battle City in Unity Part 8
summary: This post is part 8 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

### The real deal starts here

Now is time to do some damage. Destroying bricks or Steel Wall are no fun since they are lifeless, but defeating the enemy brings satisfaction. Surely you can hear your computer groaning when you beat it in a game or the evil grinning sound it makes when you go Game Over. This is the real deal. Let's get started!

### Dealing Damage

What do you mean by dealing damage? The tanks in this game are weak; it gets destroyed once hit by a CanonBall. Well, you are only partially right. There are instances where tanks do not get killed with a single shot in Battle City. For example, when a tank gets spawned, there will be an electric aura around it where it is "invincible",  so technically you are dealing damage to it, just that it is not significant to kill it. Or another instance when the armored tank comes into the picture, you will need to strike it 4 times before it gets brought down so we can say that it can take 4 damage. 

What I am going to do is to fulfill all the single damage, 4 hit damage and invincibility all in one by letting it take damage.

### How to take damage

Start by creating a new script under SmallTank GameObject calling it <keyword>Health</keyword>. Add two new integer variable called <keyword>actualHealth</keyword> and <keyword>currentHealth</keyword> to the script, exposing actualHealth to the Inspector using the SerializeField. 

actualHealth is the health that the tank should have at the beginning. So all types of tanks should have health value of 1 other than the Armored Tank which has 4. We will default that to 1 and will change it in the prefab later if required. 

currentHealth is how much health the tank has currently so an armored tank that is shot at twice will have its currentHealth as 2.
{% highlight csharp%}
[SerializeField]
int actualHealth = 1;
int currentHealth;
{% endhighlight %}

Create a new public routine called <keyword>TakeDamage</keyword>. What we will do is to deduct the currentHealth by 1 and check if currentHealth is less than or equal 0, if yes, trigger a routine called Death.

{% highlight csharp%}
public void TakeDamage()
{
    currentHealth--;
    if (currentHealth <= 0)
    {
		//trigger Death routine
    }
}
{% endhighlight %}

Create two public routines called <keyword>SetHealth</keyword> and <keyword>SetInvincible</keyword>. For SetHealth, its task is to set the value of currentHealth to the value of actualHealth, and for SetInvincible we will set the value of currentHealth to 1000. Add SetHealth as a routine to run in the Start Monobehaviour so that the tank gets assigned its proper health points at the start. 

<div class="info">Note what I am doing for invincibility. Rather than figuring out more code to make it such that it will not take damage, I just set the damage tolerance to an impossible level, so it does not die while in invincible. Short and sweet and case closed.</div>

{% highlight csharp%}
void Start()
{
    SetHealth();    
}
public void SetHealth()
{
    currentHealth = actualHealth;
}

public void setInvincible(){
	currentHealth = 1000;
}
{% endhighlight %}


Now let's create the code for the <keyword>Death</keyword> routine. This routine does not need to be exposed outside of the class. First thing we need to think of a way to identify if this is a Player Tank or an Enemy Tank. 

The best way is to create a <keyword>tag</keyword> for each type. The Player Tank will be assigned a "<keyword>Player</keyword>" tag and Enemy tank will be assigned with "<keyword>Small</keyword>", "<keyword>Fast</keyword>", "<keyword>Big</keyword>" and "<keyword>Armored</keyword>" tag baed on their types. There is already a Player tag by default so we need to create the tags for the enemies. 

We will leave the tagging till we finished creating our tank and duplicate it to make the rest of the tank types. For now, we will just take note of the Player and Enemy tag.

We will do a <keyword>gameObject.CompareTag</keyword> to check if the tank is player or enemy. If it is a player, it will trigger to spawn a player tank.

{% highlight csharp%}
void Death(){
    if (gameObject.CompareTag("Player"))
    {
            //Spawn Player
    }
}
{% endhighlight %}

If it is the enemy, do a comparetag for each of the tag and add one to the static int in MasterTracker for the type of tank destroyed.

{% highlight csharp%}
if (gameObject.CompareTag("Small")) MasterTracker.smallTankDestroyed++;
else if (gameObject.CompareTag("Fast")) MasterTracker.fastTankDestroyed++;
else if (gameObject.CompareTag("Big")) MasterTracker.bigTankDestroyed++;
else if (gameObject.CompareTag("Armored")) MasterTracker.armoredTankDestroyed++;
{% endhighlight %}

Once done, destroy the Game Object. So the consolidated code for the Heath script will be as below. The Spawn Player and Gameover will be handled in other posts when we talk about spawning the tank and GameManager. Also update the TakeDamage routine replacing the trigger death routine comment with Death routine.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Health : MonoBehaviour {
    [SerializeField]
    int actualHealth;
    int currentHealth;
    void Start()
    {
        SetHealth();    
    }	
    public void TakeDamage()
    {
        currentHealth--;
        if (currentHealth <= 0)
        {
            Death();
        }
    }
    public void SetHealth()
    {
        currentHealth = actualHealth;
    }
    public void SetInvincible()
    {
        currentHealth = 1000;
    }
    void Death()
    {
        if (gameObject.CompareTag("Player"))
        {
            //Spawn Player
        }else{
            if (gameObject.CompareTag("Small")) MasterTracker.smallTankDestroyed++;
            else if (gameObject.CompareTag("Fast")) MasterTracker.fastTankDestroyed++;
            else if (gameObject.CompareTag("Big")) MasterTracker.bigTankDestroyed++;
            else if (gameObject.CompareTag("Armored")) MasterTracker.armoredTankDestroyed++;
        }
        Destroy(gameObject);
    }
}
{% endhighlight %}


### Triggering TakeDamage

Now the script for <keyword>Health</keyword> is completed, we can head back to the <keyword>Projectile</keyword> script to add in the code to damage the tanks. Go to the routine for <keyword>onCollisionEnter2D</keyword>, check if the collision gameObject has component "Health". If yes, trigger TakeDamage. 
<div class="info">We can do it this way because only the Tanks contain the Health component so by checking if the collision Game Object  has component Health, we can safely deduce if the collision GameObject is a tank.</div>

{% highlight csharp%}
if (collision.gameObject.GetComponent<Health>() != null)
{
    collision.gameObject.GetComponent<Health>().TakeDamage();
}
{% endhighlight %}

### Summary and Full Code

Now let's try out our CanonBall's ability to kill a tank. I have adjusted the CanonBall's speed higher so it does not look anti-climatic. Disable Player script or EnemyAI if you have that in your tank so it cannot escape from its inevitable death.

<img src="{{ site.baseurl }}/images/BattleCity_Projectile6.gif" alt="">

With this we complete the Projectile script and its ability to damage the tanks, bricks and Steel Walls. [Next up]({{ site.baseurl }}{% post_url 2018-03-13-battle-city-unity-10 %}) is giving the tanks ability to fire. The full code for Projectile is as below.

{% highlight csharp%}
using UnityEngine;
using UnityEngine.Tilemaps;

public class Projectile : MonoBehaviour {
    public bool destroySteel = false;
    bool toBeDestroyed = false;
    GameObject brickGameObject,steelGameObject;
    Tilemap tilemap;
    public int speed = 1;
    Rigidbody2D rb2d;
	void Start () {
        rb2d = GetComponent<Rigidbody2D>();
        rb2d.velocity = transform.up * speed;
        brickGameObject = GameObject.FindGameObjectWithTag("Brick");
        steelGameObject = GameObject.FindGameObjectWithTag("Steel");
    }
    void OnEnable()
    {
        if (rb2d != null)
        {
            rb2d.velocity = transform.up * speed;
        }
    }
    void OnCollisionEnter2D(Collision2D collision)
    {
        rb2d.velocity = Vector2.zero;
        tilemap = collision.gameObject.GetComponent<Tilemap>();
        if (collision.gameObject.GetComponent<Health>() != null)
        {
            collision.gameObject.GetComponent<Health>().TakeDamage();
        }
        if ((collision.gameObject == brickGameObject) || (destroySteel && collision.gameObject == steelGameObject))
        {
            Vector3 hitPosition = Vector3.zero;
            foreach (ContactPoint2D hit in collision.contacts)
            {
                hitPosition.x = hit.point.x - 0.01f * hit.normal.x;
                hitPosition.y = hit.point.y - 0.01f * hit.normal.y;
                tilemap.SetTile(tilemap.WorldToCell(hitPosition), null);
            }
        }
        //keep the projectile inactive if hit anything. this will allow the projectile to be reused instead of wasting resource for garbage collector to clear it from memory
        this.gameObject.SetActive(false);
    }
    void OnDisable()
    {
        if (toBeDestroyed)
        {
            Destroy(this.gameObject);
        }
    }
    //function called from Tank to destroy the projectile when the tank is destroyed
    public void DestroyProjectile()
    {
        //if the projectile is already inactive, destroy the projectile gameobject
        if (gameObject.activeSelf == false)
        {
            Destroy(this.gameObject);
        }
        //set flag toBeDestroyed so that if projectile is still active checking the flag toBeDestroyed during onDisable to destroy the projectile
        toBeDestroyed = true;
    }
}
{% endhighlight %}
