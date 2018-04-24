---
layout: post
title: "Recreate Battle City in Unity Part 13 : The Gamemaker - Spawning"
excerpt: Recreate Battle City in Unity Part 13
summary: This post is part 13 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

One of the critical roles of the GamePlayManager is to trigger the Spawning of enemies and Player(The actual spawning is done by the spawn point). Being the manager means it gets to choose where and when to spawn. To be able to decide where to spawn, it will need to know where are the spawn points. 

Let's start with the prefab of SpawnPoint and PlayerSpawnPoint which we created at the end of [Recreate Battle City in Unity Part 10: The Spawn]({{ site.baseurl }}{% post_url 2018-03-14-battle-city-unity-11 %}). Create and assign a tag for the SpawnPoint prefab called <keyword>EnemySpawnPoint</keyword>. Create and assign a tag for the PlayerSpawnPoint prefab called <keyword>PlayerSpawnPoint</keyword>.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_1.png" alt="">

Now let's get back to the <keyword>GamePlayManager</keyword>. Declare 2 GameObject array called <keyword>spawnPoints, spawnPlayerPoints</keyword>. Go the <keyword>Start</keyword> Monobehaviour to get references to all the SpawnPoints in the Scene(there's none yet, we will add them later) using <keyword>GameObject.FindGameObjectsWithTag("EnemySpawnPoint")</keyword>. Do the same for PlayerSpawnPoint looking for the tag PlayerSpawnPoint.

{% highlight csharp%}
GameObject[] spawnPoints, spawnPlayerPoints;
void Start()
{
spawnPoints = GameObject.FindGameObjectsWithTag("EnemySpawnPoint");
spawnPlayerPoints = GameObject.FindGameObjectsWithTag("PlayerSpawnPoint");
//earlier codes are omitted for focus
}
{% endhighlight %}

### How to trigger the Spawn
Now we have the references to the Spawn Points. Let's write the code for triggering the spawning.

We start off with the spawning of enemies. Create a routine <keyword>SpawnEnemy</keyword>. The 1st thing we need to do is to check if there are any more available tanks for us to spawn. We can do that by counting the number of all the tank types which we made static in <keyword>LevelManager</keyword>(smallTank, fastTank, bigTanks, armoredTanks). If it is more than 0, then we can spawn.

{% highlight csharp%}
if (LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks > 0)
{% endhighlight %}

Next, we need to choose the SpawnPoint. We do that by randomize choosing one of the elements of the array of SpawnPoint GameObject and then trigger the Animator variable <keyword>Spawning</keyword>

<div class="info">We set a trigger that will start the StarTwinkle animation which at the end will instantiate the tank prefab in the post for The Spawn</div>

{% highlight csharp%}
public void SpawnEnemy()
{
    if (LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks > 0)
    {
        int spawnPointIndex = Random.Range(0, spawnPoints.Length);
        Animator anime = spawnPoints[spawnPointIndex].GetComponent<Animator>();
        anime.SetTrigger("Spawning");
    }
}
{% endhighlight %}
Now let's test it out! 
* Set value of 1 to the smallTankInThisLevel in the LevelManager Component of Canvas

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_2.png" alt="">

* Drag and drop some SpawnPoint prefabs onto the Hierarchy. Make sure it is within the game play area.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_3.gif" alt="">

* Add the SpawnEnemy routine to the StartStage coroutine ensuring it is triggered after the curtain raiser sequence by adding a yield return null
{% highlight csharp%}
IEnumerator StartStage()
{
    StartCoroutine(RevealStageNumber());
    yield return new WaitForSeconds(5);
    StartCoroutine(RevealTopStage());
    StartCoroutine(RevealBottomStage());
    yield return null;
    SpawnEnemy();
}
{% endhighlight %}

* Update the Start Monobehaviour as below.
{% highlight csharp%}
void Start () 
{
    spawnPoints = GameObject.FindGameObjectsWithTag("EnemySpawnPoint");
    spawnPlayerPoints = GameObject.FindGameObjectsWithTag("PlayerSpawnPoint");
    StartCoroutine(StartStage());
}
{% endhighlight %}

* Enable back all the Canvas UI elements if you have disabled it earlier. Now start playing! Play it a few times to ensure it spawns randomly at different Spawn Points.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_4.gif" alt="">

### Continuous Spawning

So now we can spawn, but we need to keep spawning until we run out of enemies. We will be using a routine called InvokeRepeating to the SpawnEnemy routine.

Let's update the StartStage coroutine to make the changes.
{% highlight csharp%}
IEnumerator StartStage()
{
    StartCoroutine(RevealStageNumber());
    yield return new WaitForSeconds(5);
    StartCoroutine(RevealTopStage());
    StartCoroutine(RevealBottomStage());
    yield return null;
    InvokeRepeating("SpawnEnemy", 5, 5); //changed to InvokeRepeating
}
{% endhighlight %}

<div class="info">The invokeRepeating takes in two more parameters outside of the routine it is calling. The 1st is for when to start the invoke, 2nd is at what frequency to start the repeat. In my case, it will call SpawnEnemy after 5 seconds and repeat the invoke every 5 seconds.</div>

I decided to add set the Spawn frequency as a Static variable from LevelManager so that we can change it from the LevelManager the speed of the spawn.

So update the LevelManager Script as below.

{% highlight csharp%}
[SerializeField]
int smallTanksInThisLevel, fastTanksInThisLevel, bigTanksInThisLevel, armoredTanksInThisLevel, stageNumber;
public static int smallTanks, fastTanks, bigTanks, armoredTanks;
[SerializeField]
float spawnRateInThisLevel=5;       //newly added
public static float spawnRate { get; private set; } //newly added
private void Awake()
{
    MasterTracker.stageNumber = stageNumber;
    smallTanks = smallTanksInThisLevel;
    fastTanks = fastTanksInThisLevel;
    bigTanks = bigTanksInThisLevel;
    armoredTanks = armoredTanksInThisLevel;
    spawnRate = spawnRateInThisLevel; //newly added
}
{% endhighlight %}

Then update the InvokeRepeating for SpawnEnemy to the below.

{% highlight csharp%}
InvokeRepeating("SpawnEnemy", LevelManager.spawnRate, LevelManager.spawnRate);
{% endhighlight %}

### Stop Spawning

Once the enemy exhausted its reserve of tanks, we will need to stop the spawning. We will do this in the SpawnEnemy routine itself where we have an "if" statement to see if there are any more enemy tanks left. We will add a CancelInvoke command to the "else" portion of the statement.
{% highlight csharp%}
void SpawnEnemy()
{
    if (LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks > 0)
    {
        int spawnPointIndex = Random.Range(0, spawnPoints.Length);
        anime = spawnPoints[spawnPointIndex].GetComponent<Animator>();
        anime.SetTrigger("Spawning");
    }
    else
    {
        CancelInvoke();
    }
}
{% endhighlight %}

### Which enemy tank type to spawn

We can now spawn tanks continuously. But each stage has their own varying number of different tank types to spawn. If LevelManager states that there are only 10 small tanks, we must be able to stop spawning small tanks. Instead, we should be spawning the other types that still have in reserve. 

The honor of doing this job will be handled by the Spawner itself as it is the one picking from random which tank to spawn. 

We will be using something similar to the RandomDirection routine where we add the direction that has no blocking to a list before asking the computer to pick 1 from the list randomly . 

This time we will add the type of tank that still has quantity in reserve based on the number of each of the tank type left to be spawned to a list(i called it <keyword>tankToSpawn</keyword>).

So let's head to the Spawner script. First, we need to have an enum with each tank type for easy identification.

{% highlight csharp%}
enum tankType
{
    smallTank, fastTank, bigTank, armoredTank
};
{% endhighlight %}

<div class="info">The enum is just to identify from the code what we are doing. All we want is actually the index number which needs to correspond with the index in tanks array in Spawner script. You can just use numbers 0-3 but it will mean you need to remember the sequence of the tank types</div>

Then we can check with the <keyword>LevelManager</keyword> to see if each tank type has more than 0 tanks in its reserve before adding it to the list(tankToSpawn) for the computer to randomly pick. Add the below code to SpawnNewTank routine in Spawner script.

{% highlight csharp%}
List<int> tankToSpawn = new List<int>();
tankToSpawn.Clear();
if (LevelManager.smallTanks > 0) tankToSpawn.Add((int)tankType.smallTank);
if (LevelManager.fastTanks > 0) tankToSpawn.Add((int)tankType.fastTank);
if (LevelManager.bigTanks > 0) tankToSpawn.Add((int)tankType.bigTank);
if (LevelManager.armoredTanks > 0) tankToSpawn.Add((int)tankType.armoredTa
{% endhighlight %}

Then we can ask Skynet to randomly pick from the list the tank type it still can spawn.
{% highlight csharp%}
int tankID = tankToSpawn[Random.Range(0, tankToSpawn.Count)];
tank = Instantiate(tanks[tankID], transform.position, transform.rotation);
{% endhighlight %}

Of course, we will later need to deduct the tank spawned from its reserve.

{% highlight csharp%}
if (tankID == (int)tankType.smallTank) LevelManager.smallTanks--;
else if (tankID == (int)tankType.fastTank) LevelManager.fastTanks--;
else if (tankID == (int)tankType.bigTank) LevelManager.bigTanks--;
else if (tankID == (int)tankType.armoredTank) LevelManager.armoredTanks--;
{% endhighlight %}

We also need to ensure the above code will only be for the enemy's spawn point. So we need to check the <keyword>isPlayer</keyword> flag to determine that. If the isPlayer flag is set, we will use a simplified instantiate to spawn the first item in the tanks array. The full code for <keyword>Spawner</keyword> script up till now as below.
{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Spawner : MonoBehaviour
{
    GameObject[] tanks;
    GameObject tank;
    [SerializeField]
    bool isPlayer;
    [SerializeField]
    GameObject smallTank, fastTank, bigTank, armoredTank;
    enum tankType
    {
        smallTank, fastTank, bigTank, armoredTank
    };
    private void Start()
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
    public void StartSpawning(){
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
            if (tankID == (int)tankType.smallTank) LevelManager.smallTanks--;
            else if (tankID == (int)tankType.fastTank) LevelManager.fastTanks--;
            else if (tankID == (int)tankType.bigTank) LevelManager.bigTanks--;
            else if (tankID == (int)tankType.armoredTank) LevelManager.armoredTanks--;
        }
        else
        {
            tank = Instantiate(tanks[0], transform.position, transform.rotation);
        }
    }
    public void SpawnNewTank()
    {
        if (tank != null) tank.SetActive(true);
    }
}
{% endhighlight %}

Let's test out the spawning of enemies again. Add 3 SpawnPoint prefabs to the hierarchy, make sure they are in the gameplay area. Go to the Canvas GameObject under the Level Manager (Script) component in the Inspector and key in 3 each for small, fast, big, and armored tanks and set the Spawn Rate to 1.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_5.gif" alt="">

### What about the player?

Oops, we forget about the player. The code is relatively similar to that of SpawnEnemy. It is simpler as there is only 1 spawn point so we do not need to randomly pick a location for it. Similar to SpawnEnemy, we will do a check on MasterTracker's playerLives to ensure it is not zero before we can allow the spawning. If it is zero, then it is GAME OVER! If we trigger the spawning of player, we will also need to deduct it from the playerLives other than when it is spawning the player tank at the start of the stage. So we need to declare a boolean called <keyword>stageStart</keyword> which will be set to true at the Start Monobehaviour. Then we will check if it is the first spawn of player tank at the start of stage using the flag, if it is, we will not deduct the playerLives. The name of the routine will be called <keyword>SpawnPlayer</keyword>
{% highlight csharp%}
bool stageStart = false;
void Start(){
    stageStart = true;
    //earlier codes are omitted for focus
}

public void SpawnPlayer()
{
    if (MasterTracker.playerLives > 0)
    {
        if (!stageStart)
        {
        MasterTracker.playerLives--;
        }
        stageStart = false;
        anime = spawnPlayerPoints[0].GetComponent<Animator>();
        anime.SetTrigger("Spawning");
    }else
    {
        StartCoroutine(GameOver())
    }
}
{% endhighlight %}

Add to the StartStage Coroutine as well to spawn the player's tank at the beginning of the stage.
{% highlight csharp%}
IEnumerator StartStage()
{
    StartCoroutine(RevealStageNumber());
    yield return new WaitForSeconds(5);
    StartCoroutine(RevealTopStage());
    StartCoroutine(RevealBottomStage());
    yield return null;
    InvokeRepeating("SpawnEnemy", LevelManager.spawnRate, LevelManager.spawnRate);
    SpawnPlayer();
}
{% endhighlight %}
    


Let's head back to the Health Script's <keyword>Death</keyword> routine and update the comment SpawnPlayer with the SpawnPlayer routine from GamePlayManager.

{% highlight csharp%}
void Death()
{
    GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>(); //newly added
    if (gameObject.CompareTag("Player"))
    {
        GPM.SpawnPlayer(); //Spawn Player
    }else{
        if (gameObject.CompareTag("Small")) MasterTracker.smallTankDestroyed++;
        else if (gameObject.CompareTag("Fast")) MasterTracker.fastTankDestroyed++;
        else if (gameObject.CompareTag("Big")) MasterTracker.bigTankDestroyed++;
        else if (gameObject.CompareTag("Armored")) MasterTracker.armoredTankDestroyed++;
    }
    Destroy(gameObject);
}
{% endhighlight %}

<div class="info">The GameObject.Find and GetComponent part of the code above is to get a reference to the GamePlayManager script in the Canvas GameObject. There will only be one Game Object called Canvas in the scene so we can safely get reference this way.</div>


Now Let's try to test the game again, with a PlayerSpawnPoint as well. I will try to beat the breakneck spawn speed enemies with my 3 lives and see who lives to tell about it. Bring it on, Skynet!

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_6.gif" alt="">

Ok I lost. Long Live SkyNet! At least my script works, for a consolation.

### Fixing the Stage Number
OK I am fedup with the "New Text" word appearing at the start of stage. So let's make it show the Stage Number. Add the below to the first line of the Start Monobehaviour.
{% highlight csharp%}
stageNumberText.text = "STAGE " + MasterTracker.stageNumber.ToString();
{% endhighlight %}
Update the Canvas's LevelManager component's Stage Number to 1. And press play. Phew! One eyesore resolved.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_Spawn_7.gif" alt="">

That's all for Spawning part of GamePlayManager. Next we will move on to how do we clear the stage.