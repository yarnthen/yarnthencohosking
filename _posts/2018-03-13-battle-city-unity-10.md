---
layout: post
title: "Recreate Battle City in Unity Part 9: Firing the shots"
excerpt: Recreate Battle City in Unity Part 9
summary: This post is part 9 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

### Who's calling the shots here

We have CanonBalls that can destroy bricks, steel walls, and tanks, but that is still not enough unless you don't mind keep drag a CanonBall on to the Scene all the time. Let's now give the ability of firing shots to the tanks!

<div class="info">Most of the techniques for this post comes from <a href="https://unity3d.com/learn/tutorials/s/survival-shooter-tutorial">Unity's Survival Shooter tutorial series</a>. You can check out the tutorial if you like as it is a video tutorial available on YouTube.</div>

Spawning of canonballs needs to be from a logical position. If we were to take the tank's position to spawn the canonball, it would appear that the tank gave birth to the canonball before it starts flying off. So we need to set the point where the CanonBall gets fired; it should be shot from a certain distance in front of the tank to prevent that childbirth illusion.

Let's create a new Empty Game Object as a child of the Tank Game Object. We can call this Game Object <keyword>Gunport</keyword>. This will act as a reference point for where the CanonBall will appear when fired. Reset the transform of Gunport then move it 1 unit forward for its transform position Y. Your offset Y position might differ from mine, who knows if you are creating a Valkyria Chronicles Final Boss size tank.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_1.png" alt="">

### (Optional)No smoke without fire

Let's also create an easy animation. Have a sprite with an image of fire ready and drag it to the Hierarchy. Unity will create a Game Object for you(I said that a lot of times), name it <keyword>Fire</keyword>. Drag the Fire Game Object and make it as a child to the Gunport. Then reset its transform and disable the Fire Game Object.

What this does is that I can enable the Fire Game Object and then disable it to simulate a more realistic firing of Canon. Watch my mouse enable and disable the Fire Game Object to see what I mean.

<div class="info">I decided to change the name from Fire to FireEffect as I have a routine also named Fire. So to avoid confusion, I decide to change name.</div>

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_2.gif" alt="">

### Programming the fire sequence: Reload the CanonBall

Let's start by prefabing the <keyword>CanonBall</keyword>. We will need 2 prefabs for this. Duplicate the CanonBall Game Object in the hierarchy, naming the duplicate <keyword>EnemyCanonBall</keyword>. Drag CanonBall and EnemyCanonBall Game Object from Hierarchy to the Project Window. Unity will create prefabs for the CanonBall and EnemyCanonBall automatically. Move your prefabs to a folder of your choice for organization (Mine is called Prefabs). Once the prefabs are created, you can delete the CanonBall and EnemyCanonBall from the hierarchy.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_3.gif" alt="">

Now it's time to get your hands dirty and start coding. Let's start by creating a new C# script component under the Gunport, we will call it <keyword>WeaponController</keyword>.

We need to create a SerializeField GameObject called <keyword>projectile</keyword> to load the CanonBall/EnemyCanonBall prefab via the Inspector for the script to instantiate the canonball later. Also create a GameObject variable called <keyword>canonBall</keyword> to store the reference of an instantiated Canonball. Create a Projectile variable called <keyword>canon</keyword> to hold the reference to the projectile script in the CanonBall. We need the reference of the projectile script to trigger the DestroyProjectile command when the tank is destroyed. Lastly create an integer variable with SerializedField called <keyword>speed</keyword>. This is used to modify the speed of the projectile later.

{% highlight csharp%}
[SerializeField]
GameObject projectile;
GameObject canonBall;
Projectile canon;
[SerializeField]
int speed=1;
{% endhighlight %}

Next, we will instantiate the CanonBall, setting its position and rotation to that of Gunport and store its reference to canonBall variable. And save the reference to the Projectile script to canon. Then pass the speed value to projectile's speed variable value. We will do this in the <keyword>Start</keyword> Monobehaviour routine.

{% highlight csharp%}
void Start () 
{
    canonBall = Instantiate(projectile, transform.position, transform.rotation) as GameObject;
    canon = canonBall.GetComponent<Projectile>();
    canon.speed = speed;
}
{% endhighlight %}

We will then create a public routine called <keyword>Fire</keyword> for the code to set CanonBall to active.(Recap of the post "Creating Projectiles", the CanonBall will be set inactive once it hits something) And move the position and rotation of the CanonBall to that of the Gunport. We need to do this because the CanonBall might be in a different place and rotation as it has already been fired off and set inactive at the location where it hit an obstacle. 

We will also need to check if the CanonBall is inactive before allowing everything else to prevent firing of more than 1 shot at any time.(The CanonBall can only be fired again when it hits an obstacle and set inactive).

{% highlight csharp%}
public void Fire()
{
    if (canonBall.activeSelf == false)
    {
        canonBall.transform.position = transform.position;
        canonBall.transform.rotation = transform.rotation;
        canonBall.SetActive(true);
    }   
}
{% endhighlight %}

Lastly, let's add the code to tell when the CanonBall should destroy itself. Remember the <keyword>DestroyProjectile</keyword> routine we created in the Post for Creating Projectile? We will be using it here. We will add these to the Monobehaviour routine of <keyword>OnDestroy</keyword> which runs code when the GameObject is destroyed.

<div class="info">The Tank gets destroyed so its child the Gunport is destroyed as well.</div>

We will also add a check to make sure CanonBall is already instantiated before we run the DestroyProjectile command.

{% highlight csharp%}
void OnDestroy()
{
    if (canonBall != null) canon.DestroyProjectile();
}
{% endhighlight %}


(optional) Hmm, I almost forgot about the animation. To have the animation, we need to get the reference to the FireEffect Game Object. We will create a new GameObject Variable called fire. And assign it the reference of the FireEffect GameObject by using Transform.GetChild(0). This grabs the first child of Gunport. There is only one child(FireEffect) for Gunport so we are safe. We then create a coroutine(I'm calling it <keyword>ShowFire</keyword>) to trigger the enabling and disabling of FireEffect GameObject and trigger it in the Fire routine.

{% highlight csharp%}
GameObject canonBall, fire; //with existing code
void Start () {
    canonBall = Instantiate(projectile, transform.position, transform.rotation) as GameObject; //existing code
    canon = canonBall.GetComponent<Projectile>();//existing code
    canon.speed = speed;//existing code
    fire = transform.GetChild(0).gameObject;
}
public void Fire()
{
    if (canonBall.activeSelf == false)
    {
        canonBall.transform.position = transform.position; //existing code
        canonBall.transform.rotation = transform.rotation; //existing code
        StartCoroutine(ShowFire());
        canonBall.SetActive(true); //existing code
    }   
}
IEnumerator ShowFire()
{
    fire.SetActive(true);
    yield return new WaitForSeconds(0.3f);
    fire.SetActive(false);
}
{% endhighlight %}

That's all we need for our WeaponController. Full code as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponController : MonoBehaviour {
    [SerializeField]
    GameObject projectile;
    GameObject canonBall, fire;
    Projectile canon;
    [SerializeField]
    int speed;
    void Start () {
        canonBall = Instantiate(projectile, transform.position, transform.rotation) as GameObject;
        canon = canonBall.GetComponent<Projectile>();
        canon.speed = speed
        fire = transform.GetChild(0).gameObject;
    }
    public void Fire()
    {
        if (canonBall.activeSelf == false)
        {
            canonBall.transform.position = transform.position;
            canonBall.transform.rotation = transform.rotation;
            StartCoroutine(ShowFire());
            canonBall.SetActive(true);
        }   
    }
    IEnumerator ShowFire()
    {
    	fire.SetActive(true);
    	yield return new WaitForSeconds(0.3f);
    	fire.SetActive(false);
    }
    private void OnDestroy()
    {
        if (canonBall != null) canon.DestroyProjectile();
    }
}
{% endhighlight %}

### Giving tanks authority to fire off: Player Tank

We move back to the <keyword>Player</keyword> Script to bestow it the ability to fire off the CanonBalls! Let's determine how we can pass command to the computer for our tank to fire the shot. I decided to set the Space Bar as the key to fire the shot as it is the largest entity in the keyboard and I definitely do not want to miss taking a shot just because I press the wrong key. 

Let's first create a <keyword>WeaponController</keyword> variable(I'm calling it <keyword>wc</keyword>) to store the reference of WeaponController script in the Gunport. We save the reference in the Start Monobehaviour routine using <keyword>GetComponentInChildren</keyword> as Gunport is a child of the Tank, so we are getting the component that belongs the child of Tank Game Object.
{% highlight csharp%}
WeaponController wc;
void Start () {
    wc = GetComponentInChildren<WeaponController>();
    rb2d = GetComponent<Rigidbody2D>(); //existing code
}
{% endhighlight %}

We will use Input.GetKeyDown to detect the key pressed in the Update Monobehaviour routine. Using GetKeyDown means press and hold to fire rapidly is useless as it identifies only the moment you press down the key, we are shooting canonballs here not a machine gun! This also ensures we do not strain the Update function(I am a nice guy) to trigger the Fire command(triggered through the reference variable wc). It is useless anyway as we have coded as such in the WeaponController that you can only fire 1 shot at a time.

{% highlight csharp%}
void Update () {
    h = Input.GetAxisRaw("Horizontal"); //existing code
    v = Input.GetAxisRaw("Vertical"); //existing code
    if (Input.GetKeyDown(KeyCode.Space))
    {
        wc.Fire();
    }
}
{% endhighlight %}

That's all the code we need to let the player fire off canonballs. Now let's try it out. Add the Player Script back to the Guniea Pig Tank and disable the EnemyAI script if you have it. Also, <keyword>set the CanonBall prefab to disabled</keyword> if not the CanonBall will start firing by itself even if without your orders. Select the Gunport GameObject which is the child of your Tank Game Object and assign the <keyword>CanonBall</keyword> prefab to its <keyword>Projectile</keyword> variable in the inspector. Also, set the <keyword>Speed</keyword> to 10 so that the canonBall fly off at a pace faster than a tank.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_4.gif" alt="">

Create a new Layer called PlayerTank and assign the layer to the Guinea Pig Tank. Create a new Layer called PlayerProjectile and assign it to the Prefab of CanonBall.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_5.png" alt="">

Go to Physics2D settings in Unity (Edit -> Project Settings -> Physics2D) and make the below highlighted changes. This is to ensure the Player's CanonBall will not collide with the Player's tank or another Player's CanonBall.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_6.png" alt="">

OK. Live firing in progress, please stay out! 

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_7.gif" alt="">

That's all for Player script till now. Full Script as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : Movement {
    float h, v;
    Rigidbody2D rb2d;
    WeaponController wc;
    void Start () 
    {
        wc = GetComponentInChildren<WeaponController>();
        rb2d = GetComponent<Rigidbody2D>();
    }
    void FixedUpdate()
    {
        if (h != 0 && !isMoving) StartCoroutine(MoveHorizontal(h, rb2d));
        else if (v != 0 && !isMoving) StartCoroutine(MoveVertical(v, rb2d));
    }  
    void Update () 
    {
        h = Input.GetAxisRaw("Horizontal");
        v = Input.GetAxisRaw("Vertical");
        if (Input.GetKeyDown(KeyCode.Space))
        {
            wc.Fire();
        }
    }
}
{% endhighlight %}

### Giving tanks authority to fire off: Enemy Tank

Let's start by duplicating our Guinea Pig Tank and naming the Duplicate EnemyTank. Go to its Gunport GameObject and changing the Projectile variable in the Inspector to EnemyCanonBall.

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_8.gif" alt="">


Create 2 new layers named EnemyTank and EnemyProjectile. Assign the layer EnemyTank to EnemyTank Game Object and layer EnemyProjectile to the EnemyCanonBall prefab.
<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_9.png" alt="">

Go to Physics2D settings in Unity (<keyword>Edit -> Project Settings -> Physics2D</keyword>) and make the below highlighted changes.<br> This is to ensure the Enemy's CanonBall will not collide with the Enemy's tank. 
<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_10.png" alt="">

We move to the EnemyAI Script to give enemies the opportunity to fire off the CanonBalls! We start off by creating a WeaponController variable(I'm calling it wc) to store the reference of WeaponController script in the Gunport. We save the reference in the Start Monobehaviour routine using GetComponentInChildren as Gunport is a child of the Tank, so we are getting the component that belongs the child of Tank Game Object.

{% highlight csharp%}
WeaponController wc;
void Start () {
    rb2d = GetComponent<Rigidbody2D>(); //existing code
    RandomDirection(); //existing code
    wc = GetComponentInChildren<WeaponController>();
}
{% endhighlight %}

Next is to create the random timer to fire off. What I will do is to mimic the <keyword>Random Timer in RandomDirection</keyword>. I create a routine called <keyword>FireWhenWanted</keyword> to store the <keyword>wc.Fire()</keyword> command, then I gave instructions for it to invoke FireWhenWanted again in a <keyword>random interval between 1 second to 5 seconds</keyword>. I also add an invoke at the Start MonoBehaviour routine so that it will kick off the firing at random. We will not see the shooting of canonballs at 1-second intervals as we are protected by the code to allow one canonball to be fired at a time.

{% highlight csharp%}
void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
    RandomDirection();
    wc = GetComponentInChildren<WeaponController>();
    Invoke("FireWhenWanted", Random.Range(1f, 5f));
}
void FireWhenWanted()
{
    wc.Fire();
    Invoke("FireWhenWanted", Random.Range(1f, 5f));
}

{% endhighlight %}

### Summary

Now we are ready to try a shoot out between the Good Guy and the Bad Guys! I change my sprite with green color so I can identify myself from the enemies. I have also duplicate some more enemy tanks so that I can test my skills!

<img src="{{ site.baseurl }}/images/BattleCity_FiringTheShots_11.gif" alt="">

Great Game! But in the end I got overwhelmed by the enemies. Now <keyword>prefab your Enemy Tank</keyword> and your <keyword>Player Tank</keyword> (rename the tank to <keyword>PlayerTank</keyword> and <keyword>assign it with the tag "Player"</keyword> before prefab) as well so that you can use it for test play later. <keyword>Set both the prefab as disabled</keyword>, the prefabs will be used to instantiate the tanks later and they start out as disabled before we enable it for it to appear on the scene. That will be the end of this post. [Next post]({{ site.baseurl }}{% post_url 2018-03-14-battle-city-unity-11 %}), we will talk about spawning the enemies and player. Here is the full code for EnemyAI.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyAI : Movement {
    Rigidbody2D rb2d;
    float h, v;
    [SerializeField]
    LayerMask blockingLayer;
    WeaponController wc;
    enum Direction { Up, Down, Left, Right };
    void Start()
    {
        rb2d = GetComponent<Rigidbody2D>();
        RandomDirection();
        wc = GetComponentInChildren<WeaponController>();
        Invoke("FireWhenWanted", Random.Range(1f, 5f));
    }
    public void RandomDirection()
    {
        CancelInvoke("RandomDirection");
        List<Direction> lottery = new List<Direction>();
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, 1), blockingLayer))
        {
            lottery.Add(Direction.Right);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(0, -1), blockingLayer))
        {
            lottery.Add(Direction.Left);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(1, 0), blockingLayer))
        {
            lottery.Add(Direction.Up);
        }
        if (!Physics2D.Linecast(transform.position, (Vector2)transform.position + new Vector2(-1, 0), blockingLayer))
        {
            lottery.Add(Direction.Down);
        }
        Direction selection = lottery[Random.Range(0, lottery.Count)];
        if (selection == Direction.Up)
        {
            v = 1;
            h = 0;
        }
        if (selection == Direction.Down)
        {
            v = -1;
            h = 0;
        }
        if (selection == Direction.Right)
        {
            v = 0;
            h = 1;
        }
        if (selection == Direction.Left)
        {
            v = 0;
            h = -1;
        }
        Invoke("RandomDirection", Random.Range(3, 6));
    }
    void FireWhenWanted()
    {
        print("FireWhenWanted");
        wc.Fire();
        Invoke("FireWhenWanted", Random.Range(1f, 5f));
    }
    void OnCollisionEnter2D(Collision2D collision)
    {
        RandomDirection();
    }
    void FixedUpdate()
    {
        if (v != 0 && isMoving == false) StartCoroutine(MoveVertical(v, rb2d));
        else if (h != 0 && isMoving == false) StartCoroutine(MoveHorizontal(h, rb2d));
    }
}
{% endhighlight %}