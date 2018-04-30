---
layout: post
title: "Battle City in Unity Part 7: Creating Projectiles"
excerpt: Battle City in Unity Part 7
summary: This post is part 7 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}

### Why are you shooting at me!!!

Now we have heroes and villains ready, but all they are doing now is just walking around seeing and knocking into each other. Like all Hollywood plots, the villains should attack the hero, and the hero should defend against the villains to get to an ending. So let's give them the means to damage the other party. Let's create the canonballs for the tanks to fire at each other!

Create an empty Game Object rename it <keyword>CanonBall</keyword>. Drag the sprite for your Canonball to the Hierarchy. Unity will create Game Object for you. Move the Game Object created by Unity (when you drag in the sprite) into the CanonBall Game Object. Now reset the transform of the child Game Object and adjust its transform rotation until it is facing upwards. Mine was facing right initially, so I need to add 90 to the Transform.Rotation to its inspector. Adjust the size of your sprite by its Pixels Per Unit in Import Settings accordingly; we do not need Banzai Bill size canonballs, just size it up based on your taste. 

Add a <keyword>Box Collider 2D</keyword>. The Collider should wrap the sprite perfectly, if not, just edit its collider. Finally, add a <keyword>RigidBody 2D</keyword> to the CanonBall Game Object and set <keyword>Gravity Scale</keyword> to 0(to prevent it from falling). Set its mass to 0.01, if not we will be seeing canonballs having the ability to push a tank backward.


### Progamming the Canonball: Getting it moving

Add a script to the CanonBall Game Object calling it <keyword>Projectile</keyword>. Let's start with the obvious; the CanonBall needs to move so what we will do is <keyword>add velocity to its rigidbody 2d</keyword> so it can start moving. We will put that code in the <keyword>Start</keyword> Monobehaviour, so it starts moving when it appears on the scene. So do a getcomponent to get a reference to the rigidbody2d of the Game Object then add a velocity to it using transform.up(Remember I asked your canonball to face up? That's why!).

From watching gameplay of Battle City, there are varying speeds of the canonballs so let's add a <keyword>speed</keyword> variable as a modifier of how fast the canonball moves. We will expose it to public to allow modification from the inspector. Let's set the default speed to 1 first to see how it goes. The code will be like the below.

{% highlight csharp%}
public int speed = 1;
Rigidbody2D rb2d;
void Start () {
    rb2d = GetComponent<Rigidbody2D>();
    rb2d.velocity = transform.up * speed;
}
{% endhighlight %}

Now try to play the Game. You will see your flying Canonball(Anti-Climax! It's slow to the point that the tank is outrunning it!). So adjust the speed to your preference.

<img src="{{ site.baseurl }}/images/BattleCity_Projectile1.gif" alt="">

### Progamming the Canonball: Destroy a brick

Some items can get destroyed by the canonball. They are Tanks, Bricks, and Steel(on a more powerful tank). Let's start with the bricks. If you had followed religiously what I have done, all your bricks would be sitting on the <keyword>Bricks Tilemap</keyword> only. That makes it easy to determine we are destroying a brick. First, we need to get a reference to the Bricks Tilemap. We will do that by using <keyword>GameObject.FindGameObjectWithTag</keyword>. During the initial setup of Tilemap, we assign the tag Brick on the Bricks Tilemap, now is where it gets useful. As we are dealing with Tilemaps in the code, we will need to access the libraries related to Tilemaps, add <keyword>Using UnityEngine.Tilemaps;</keyword> before the class declaration to gain access to Tilemap related features.

Declare a GameObject Variable called <keyword>brickGameObject</keyword>. Assign the reference of Bricks TileMap (using <keyword>GameObject.FindObjectByTag("Brick")</keyword>) to brickGameObject in the Start Monobehaviour.

{% highlight csharp%}
GameObject brickGameObject; //added code for Destroy a brick

public int speed = 1;
Rigidbody2D rb2d;
 void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
    rb2d.velocity = transform.up * speed;
    brickGameObject = GameObject.FindGameObjectWithTag("Brick"); //added code for Destroy a brick
}
{% endhighlight %}

Add Monobehaviour's <keyword>OnCollisionEnter2D</keyword> to write in the code to design how the CanonBall will react when colliding. We will need to determine if the Game Object colliding with the CanonBall is a brick. Thankfully the OnCollisionEnter2D pumps in a parameter of <keyword>Collision2D</keyword> which takes in the information of what has clashed with the CanonBall so we can use that to compare with the brickGameObject to determine if that is a brick. We will also need to stop the CanonBall once it hit a collider as it is not expected to move anymore. The code will be similar to below.

{% highlight csharp%}
private void OnCollisionEnter2D(Collision2D collision)
{
	rb2d.velocity = Vector2.zero; //stopping the CanonBall upon impact
    if (collision.gameObject == brickGameObject) {
        //the action taken if the collided object is a brick
    }
}
{% endhighlight %}

Now with the means to detect if it is a brick, let's add the code for destroying the brick. Add in the UnityEngine.Tilemaps library. We need to declare a tilemap variable to store the tilemap component of the collision Game Object before using it as a reference to destroying the tile on the collision position on the tilemap.

{% highlight csharp%}
using UnityEngine.Tilemaps;
public class Projectile : MonoBehaviour {
Tilemap tilemap; // added code for tilemap variable
//earlier code are omitted for focus
private void OnCollisionEnter2D(Collision2D collision)
{
	rb2d.velocity = Vector2.zero;
    tilemap = collision.gameObject.GetComponent<Tilemap>();// added code for tilemap variable
    if (collision.gameObject == brickGameObject) {
        //the action taken if the collided object is a brick
    }
}
{% endhighlight %}

The code snippet for destroying the brick on a tilemap is from Unity's 2D Tech Demo. In summary, it is to use the contact information gathered from Collision2D to find the exact position of the contact and set the tile at that position to null(which means remove the tile and its collider at that position).

{% highlight csharp%}
Vector3 hitPosition = Vector3.zero;
foreach (ContactPoint2D hit in collision.contacts)
{
    hitPosition.x = hit.point.x - 0.01f * hit.normal.x;
    hitPosition.y = hit.point.y - 0.01f * hit.normal.y;
    tilemap.SetTile(tilemap.WorldToCell(hitPosition), null);
}
{% endhighlight %}


Gathering all the parts together, the code will be as below

{% highlight csharp%}
private void OnCollisionEnter2D(Collision2D collision)
{
	rb2d.velocity = Vector2.zero;
    tilemap = collision.gameObject.GetComponent<Tilemap>();
    if (collision.gameObject == brickGameObject)
    {
        Vector3 hitPosition = Vector3.zero;
        foreach (ContactPoint2D hit in collision.contacts)
        {
            hitPosition.x = hit.point.x - 0.01f * hit.normal.x;
            hitPosition.y = hit.point.y - 0.01f * hit.normal.y;
            tilemap.SetTile(tilemap.WorldToCell(hitPosition), null);
        }
    }
}
{% endhighlight %}


Let's try to place some brick in front of the CanonBall to see it get obliterated!

<img src="{{ site.baseurl }}/images/BattleCity_Projectile2.gif" alt="">

### Progamming the Canonball: Destroy a steel wall

The code to destroy the steel wall is the same as the ones for brick. We just need to change the reference to the <keyword>Steel Tilemap</keyword> GameObject. So this time we will need to declare a GameObject Variable called <keyword>steelGameObject</keyword>. Assign the reference of Steel TileMap (using GameObject.FindObjectByTag("Steel")) to steelGameObject in the Start Monobehaviour.

{% highlight csharp%}
GameObject brickGameObject, steelGameObject; //added code for Destroy a steel wall
public int speed = 1;
Rigidbody2D rb2d;
 void Start()
{
    rb2d = GetComponent<Rigidbody2D>();
    rb2d.velocity = transform.up * speed;
    brickGameObject = GameObject.FindGameObjectWithTag("Brick");
    steelGameObject = GameObject.FindGameObjectWithTag("Steel");
}
{% endhighlight %}

The ability of a CanonBall to destroy a Steel Wall depends upon its tank obtaining certain powerups. So we will need a way to determine if the tank has already gained the ability for its CanonBall to destroy a Steel Wall. What I will do is to declare a Public bool variable as a flag to determine if the CanonBall can annihilate the Steel Wall. I will call this <keyword>destroySteel</keyword>. In the later section when we touch on the powerups, we can then access this flag to set destroySteel to true once the ability is gained. Since the code for destroying brick and steel wall are the same, there is no point in rewriting them again; we can combine them like the below.

{% highlight csharp%}
public bool destroySteel=false;

private void OnCollisionEnter2D(Collision2D collision)
    {
    	rb2d.velocity = Vector2.zero;
        tilemap = collision.gameObject.GetComponent<Tilemap>();
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
    }
{% endhighlight %}


Let's try to place a brick and a steel wall in front of the two CanonBall. This time the destroySteel is set to false. I will make the CanonBalls hit the brick and the steel wall. You will see that the brick will be gone, but the steel wall is still standing, laughing back at you.

<img src="{{ site.baseurl }}/images/BattleCity_Projectile3.gif" alt="">

Now let's set destroySteel to true. You will see both the brick and steel wall get wiped out! So who is having the last laugh now? Ha ha ha!

<img src="{{ site.baseurl }}/images/BattleCity_Projectile4.gif" alt="">

### Progamming the Canonball: Destroy itself

The last part of this post is to destroy the CanonBall itself. All this while, we are seeing the CanonBall obliterating things in its path and merely stop if it hits something it cannot kill. We are not creating some indestructible armament here, so we will need to destroy the CanonBall upon contact. 

Well, nearly. Instead what we are going to do is to disable the CanonBall upon collision. We will only disable it so that it can be recycled for use again for the next shot by the tank. 

<div class="info">Destroying GameObjects results in Garbage Collection to housekeep memory in Unity which takes up resources, so more GameObjects destroyed more Garbage collection resources required. So this is performance optimization that we are doing here. To know more, you can read <a href="https://unity3d.com/learn/tutorials/topics/performance-optimization/optimizing-garbage-collection-unity-games">Unity's article on Garbage Collection</a></div>

Recycling the CanonBall for usage is also adding to the challenge of this game as it will mean every tank can only fire a single shot at one time.

Of course, the CanonBall still need to be destroyed(when its tank get destroyed so there is no chance it will get recycled again) but we can reduce the number of Game Objects destroyed significantly.

The first part to destroy the CanonBall is straightforward. Just the below code at the end of the OnCollisionEnter2D Monobehaviour routine. The routine will only set the Game Object to inactive.

{% highlight csharp%}
this.gameObject.SetActive(false);
{% endhighlight %}

The next part is how to recycle the CanonBall. Since what we are doing is to disable it, so to reuse it is by enabling it. The enabling back will be done by the Tank which will be firing the shot so what the Projectile Script needs to take care is how the CanonBall should behave when enabled again. 

To achieve that, we can use the Monobehaviour Routine <keyword>OnEnable</keyword> which allows us to give instructions when the Game Object is enabled. All we will need to do is add back the velocity to the CanonBall for it to start flying again.

{% highlight csharp%}
void OnEnable()
{
    rb2d.velocity = transform.up * speed;
}
{% endhighlight %}

The last part will be when to destroy the CanonBall. We will need to destroy the CanonBall when its tank got destroyed. We will create a Public Routine called <keyword>DestroyProjectile</keyword> to handle that. The Tank will trigger the routine to the CanonBall once it gets destroyed so the CanonBall can know its time is up.

{% highlight csharp%}
public void DestroyProjectile()
{
    Destroy(this.gameObject);
}
{% endhighlight %}


But wait, there’s more. What if a CanonBall is flying halfway and its tank got destroyed? We cannot disintegrate a CanonBall halfway in its flight just because its tank got destroyed. Let's be fair to the canonball!

To overcome it, let’s use a boolean flag I'm naming <keyword>toBeDestroyed</keyword>, so if a CanonBall receives a command to self-destruct, set toBeDestroyed to true. The DestroyProjectile routine will check if the CanonBall is disabled before it destroys itself. If it is still enabled, it will set the toBeDestroyed flag to true.

In the Monobehaviour routine <keyword>onDisable</keyword>, there will be a check to see if toBeDestroyed is set to true. If yes, the CanonBall will self-destruct as it is already disabled.

{% highlight csharp%}
bool toBeDestroyed = false;

void OnDisable()
{
    if (toBeDestroyed)
    {
        Destroy(this.gameObject);
    }
}


public void DestroyProjectile()
{
    if (gameObject.activeSelf == false)
    {
        Destroy(this.gameObject);
    }
    toBeDestroyed = true;
}
{% endhighlight %}


Now let’s test our mini-Banzai Bill again! I deliberately expose the toBeDestroyed flag, so we can see the effect of setting it. You can see that CanonBall1 which has its toBeDestroyed unchecked will be disabled, but CanonBall2 whose toBeDestroyed flag is true gets destroyed.

<img src="{{ site.baseurl }}/images/BattleCity_Projectile5.gif" alt="">


<div class="info">PostMortem notes: While testing out the mini-Banzai Bill, I noticed a bug in the onEnabled routine. There should be an additional check to see if rb2d is null before setting its velocity. This is because the OnEnable command runs before the Start Routine which is the one setting the reference to the rigidbody2d component for rb2d so the onEnabled command will fail as the rb2d is not initialized yet. So we need to update the OnEnable to the below.</div>

{% highlight csharp%}
private void OnEnable()
{
    if (rb2d != null)
    {
        rb2d.velocity = transform.up * speed;
    }
}
{% endhighlight %}


### Conclusion and Full Code

Phew! Finally, reach the end of this post. We are not done with CanonBall yet. We have yet to touch on destroying the tanks. I will leave that to the [next post]({{ site.baseurl }}{% post_url 2018-03-12-battle-city-unity-9 %}) as it involves touching on both the Tank Game Object and CanonBall Game Object. Here's the full code for Projectile up till now.

{% highlight csharp%}
using UnityEngine;
using UnityEngine.Tilemaps;

public class Projectile : MonoBehaviour {
    public bool destroySteel = false;
    [SerializeField]
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
    private void OnEnable()
    {
        if (rb2d != null)
        {
            rb2d.velocity = transform.up * speed;
        }
    }
    private void OnCollisionEnter2D(Collision2D collision)
    {
        rb2d.velocity = Vector2.zero;
        tilemap = collision.gameObject.GetComponent<Tilemap>();
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
    private void OnDisable()
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
{% include serieswithintro.html %}
