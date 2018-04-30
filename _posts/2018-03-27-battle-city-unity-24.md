---
layout: post
title: "Battle City in Unity Part 23 : Bonus Crates - Stopwatch"
excerpt: Battle City in Unity Part 23
summary: This post is part 23 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}

The stopwatch is the hardest for me to figure out of the bonus crates effects. The additional things we need to add on is not a lot, but somehow all the ways I tried did not seem to work. Probably someone can later comment on how it can be done better as my way does not appear as straightforward. The effects of the stopwatch are to freeze all the enemy tanks for a period.

Start by dragging and dropping the Sprite you have for stopwatch bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>Stopwatch</keyword>.Add a <keyword>Box Collider 2D</keyword> and assure its boundary fills the entire crate's image, then make it a trigger. Also, assign <keyword>Powerups</keyword> layer to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_Stopwatch_1.png" alt="">

### Creating the freezing effect

Same as the Grenade, the key to this effect is to be able to find all the enemy tanks on the scene. Fortunately, we have already done the hard work at the last post to find all the enemy tanks, so we need just to reuse it.

Now we will create the code for the Stopwatch. Add a new script called <keyword>Stopwatch</keyword> to the Stopwatch Game Object. Set the class to inherit from PowerUps. The full code for Stopwatch as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Stopwatch : PowerUps
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
        GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
        GPM.ActivateFreeze();
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

<div class="info">The code for Stopwatch is minimal as all it does is to call a routine in GamePlayManager. Most of the hard work will be done by the GamePlayManager. The freezing effect will only last for about 10 seconds, so this will need to be done via a coroutine. We cannot destroy the object before the coroutine completes so the best way is to pass the job to GamePlayManager to do it.</div>

Through trial and error, I have realized the only way to freeze the enemy tanks is through the following sequence:
1. Disable the tank Game Object.
2. Stop coroutine and invoke triggered from the EnemyAI script.
3. Disable the EnemyAI component.
4. Enable the tank Game Object.

Just disabling the EnemyAI component won't work as it will get enabled back by itself during runtime(I am not sure why this is happening). 

To overcome this, I need to disable the GameObject first. The stopping of coroutine and invoke is to prevent the tank from moving or shooting. Then we can disable the EnemyAI safely before enabling back the Game Object for it to be visible. 

There are 2 types of freezing scenario; one is the freezing of enemy tanks in the gameplay area, the other is the freezing of the enemy tanks that got spawned during the freezing timeline. The freezing of enemy tanks in the gameplay area will be handled by the GamePlayManager while the Spawner script will handle those that get spawned during the freeze period.

The stopping and resumption of invoke and coroutine will be handled inside the EnemyAI script becausing the cancelling of invoke and coroutine triggered by <keyword>EnemyAI</keyword> can only be done inside the EnemyAI script. 

Create 2 routines <keyword>ToFreezeTank</keyword> and <keyword>ToUnfreezeTank</keyword>. ToFreezeTank will do the stopping and ToUnfreezeTank will do the resumption. We will also need a public static boolean called <keyword>freezing</keyword> which will use by the Spawner to check if there is a need to freeze a newly spawned tank.

{% highlight csharp%}
public static bool freezing= false;
public void ToFreezeTank()
{
    CancelInvoke();
    StopAllCoroutines();
}
public void ToUnfreezeTank()
{
    isMoving = false;
    RandomDirection();
    Invoke("FireWhenWanted", Random.Range(0.5f, 1));     
}
{% endhighlight %}

For those tanks in the gameplay area, add the below code to the GamePlayManager script.

{% highlight csharp%}
Transform enemyHolder;
public void ActivateFreeze()
{
    StartCoroutine(FreezeActivated());
}

IEnumerator FreezeActivated()
{
    EnemyAI.freezing = true;
    enemyHolder = GameObject.Find("EnemyHolder").transform;
    for (int i = 0; i < enemyHolder.childCount; i++)
    {
        enemyHolder.GetChild(i).gameObject.SetActive(false);
        enemyHolder.GetChild(i).gameObject.GetComponent<EnemyAI>().ToFreezeTank();
        enemyHolder.GetChild(i).gameObject.GetComponent<EnemyAI>().enabled = false;
        enemyHolder.GetChild(i).gameObject.SetActive(true);
    }
    yield return new WaitForSeconds(10);
    for (int i = 0; i < enemyHolder.childCount; i++)
    {
        enemyHolder.GetChild(i).gameObject.SetActive(false);
        enemyHolder.GetChild(i).gameObject.GetComponent<EnemyAI>().enabled = true;
        enemyHolder.GetChild(i).gameObject.GetComponent<EnemyAI>().ToUnfreezeTank();
        enemyHolder.GetChild(i).gameObject.SetActive(true);
    }
    EnemyAI.freezing = false;
}
{% endhighlight %}
<div class="info">The first routine <b>ActivateFreeze</b> is simply for triggering the coroutine <b>FreezeActivated</b>. As the task needs to run over a period, the best is to do via a coroutine.<br><br>
ActivateFreeze might appear redundant but it is required as <b>StopWatch</b> script cannot trigger the coroutine as the script component will get destroyed before the coroutine completes which will mean the coroutine will get canceled. So we can only get StopWatch to trigger a routine in <b>GamePlayManager</b> which will in turn trigger a coroutine.<br><br>
The first half of the coroutine is to do disable of <b>EnemyAI</b> component and stopping of the coroutines and Invoke triggered by EnemyAI. Then there will be a wait for 10 seconds before enabling back EnemyAI and starting back the coroutine and invoke by EnemyAI script. The static boolean <b>freezing</b> will be set to true at the start of the freezing period and back to false at the end.
</div>

We now head to the Spawner script to add in the portion of freezing the tank when spawned. This will be added at the SpawnNewTank routine as the tank Game Object will be set active then. The code is similar to the first half of FreezeActivated coroutine. There is no need to do the unfreeze as it will be handled by the second half of the FreezeActivated coroutine which will check for all the child of EnemyHolder and activate all their EnemyAI component.
{% highlight csharp%}
public void SpawnNewTank()
{
    if (tank != null)
    {
        tank.SetActive(true);
        if (EnemyAI.freezing == true)
        {
            tank.SetActive(false);
            tank.GetComponent<EnemyAI>().ToFreezeTank();
            tank.GetComponent<EnemyAI>().enabled = false;
            tank.SetActive(true);
        }
    }
}   
{% endhighlight %}

Now let's test it! 

<img src="{{ site.baseurl }}/images/BattleCity_Stopwatch_2.gif" alt="">

Mission accomplished. [Next up]({{ site.baseurl }}{% post_url 2018-03-28-battle-city-unity-25 %}) is the Level up Bonus Crate.
{% include serieswithintro.html %}