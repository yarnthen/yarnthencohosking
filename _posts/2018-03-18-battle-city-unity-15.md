---
layout: post
title: "Recreate Battle City in Unity Part 14 : The Gamemaker - Stage Cleared or Game Over"
excerpt: Recreate Battle City in Unity Part 14
summary: This post is part 14 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

Let's look at what happened during Game Over or Stage Clear.(with fast fwd enabled)

<img src="{{ site.baseurl }}/images/BattleCity_StageClear_GameOver_1.gif" alt="">

We can see that both Stage Clear and Game Over goes to the same scene for Score Calculation but Game Over shows a massive Game Over sign after while the Stage Clear goes to next stage. So there must be a means to tell the Score Calculation Scene that it is a Game Over or a Stage Clear scenario. 

The best way to do that will be at the <keyword>MasterTracker</keyword> as that is the only object that will persist between scenes and thus the only candidate to pass the information. All we need to do is to add a <keyword>public static bool</keyword> variable called <keyword>stageCleared</keyword> in MasterTracker. If it is a stage cleared scenario, GamePlayManager can set the stageCleared to Yes, if not, it will be false.

Add the below to the MasterTracker's variable declaration

{% highlight csharp%}
public static bool stageCleared = false;
{% endhighlight %}

We can now go to the <keyword>GamePlayManager</keyword> to code in the part of transiting to the next scene. We will add another bool called <keyword>tankReserveEmpty</keyword> where it will be set to true when all available enemy tanks are spawned. We will set this to true in the SpawnEnemy routine where we check if there are any more enemy tanks left to spawn.

{% highlight csharp%}
bool tankReserveEmpty = false;
void SpawnEnemy()
{
    if (LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks > 0)
    {
        int spawnPointIndex = Random.Range(0, spawnPoints.Length);
        Animator anime = spawnPoints[spawnPointIndex].GetComponent<Animator>();
        anime.SetTrigger("Spawning");
    }
    else
    {
        CancelInvoke();
        tankReserveEmpty = true;
    }
}
{% endhighlight %}

Then at the Update Monobehaviour routine, we can check if the tankReserveEmpty flag is set and there are no more enemy tanks in the scene(using GameObject.FindWithTag). If the check is true, set MasterTracker's stageCompleted to true and trigger a routine <keyword>LevelCompleted</keyword> to move to the Score Calculation scene(which I will call Score). We will also reset the tankReserveEmpty flag back to false. We will need to add <keyword>using UnityEngine.SceneManagement</keyword> as we need its static routine(<keyword>LoadScene</keyword>) to go to the next scene.
{% highlight csharp%}
using UnityEngine.SceneManagement;

private void Update()
{
    if (tankReserveEmpty && GameObject.FindWithTag("Small") == null && GameObject.FindWithTag("Fast") == null && GameObject.FindWithTag("Big") == null && GameObject.FindWithTag("Armored") == null)
    {
    	MasterTracker.stageCleared = true;
        LevelCompleted();
    }
}
private void LevelCompleted()
{
    tankReserveEmpty = false;
    SceneManager.LoadScene("Score");
}
{% endhighlight %}

We can also code in the GameOver portion at the GameOver coroutine. The only update we need to do is set MasterTracker's stageCleared to false and trigger the LevelCompleted routine at the end.

{% highlight csharp%}
IEnumerator GameOver()
{
    while (gameOverText.rectTransform.localPosition.y < 0)
    {
        gameOverText.rectTransform.localPosition = new Vector3(gameOverText.rectTransform.localPosition.x, gameOverText.rectTransform.localPosition.y + 40f * Time.deltaTime, gameOverText.rectTransform.localPosition.z);
        yield return null;
    }
    MasterTracker.stageCleared = false;
    LevelCompleted();
}
{% endhighlight %}

### Game Over when Eagle is destroyed

There is still one more Game Over scenario left: When the eagle is destroyed. Let's head to the script <keyword>Eagle</keyword> which we created at the end of the post [Level Creation using Tilemaps]({{ site.baseurl }}{% post_url 2018-03-06-battle-city-unity-3 %}). Add the onCollisionEnter2D routine to the script with the below code.

{% highlight csharp%}
private void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.CompareTag("EnemyProjectile") || collision.gameObject.CompareTag("PlayerProjectile"))
    {
        GetComponent<SpriteRenderer>().enabled = false;
        transform.GetChild(0).gameObject.SetActive(true);
        GamePlayManager GPM = GameObject.Find("Canvas").GetComponent<GamePlayManager>();
        StartCoroutine(GPM.GameOver());
    }

}
{% endhighlight %}

<div class="info">In the above code, we check for what is colliding with the eagle before deciding if it is Game Over. Only the canonballs can destroy the Eagle, so we do a comparetag to ensure the colliding party is a canonball(friend or foe) before triggering the Game Over. What we did to simulate the destruction of the eagle is to set the SpriteRenderer of the Eagle to disabled and to enable the broken flag game object that is child to the Eagle and set inactive initially. We then need to tell GamePlayManager to trigger the Game Over sequence by starting the coroutine GameOver</div>.

We need to assign tags to the <keyword>CanonBall</keyword> and <keyword>EnemyCanonBall</keyword> prefabs so that we can identify it in the code. So create 2 new tags <keyword>PlayerProjectile</keyword> and <keyword>EnemyProjectile</keyword> and assign them to CanonBall and EnemyCanonBall respectively.

Let's test out the Eagle Destruction Game Over Scenario now!

<img src="{{ site.baseurl }}/images/BattleCity_StageClear_GameOver_2.gif" alt="">

Hooray! It's working, with some error at the end as it cannot find the Score Scene stated in the Code. We will create the Score Scene in the next post.<keyword>Rename the Stage you are in as Stage1</keyword>. Before we leave let's do some housekeeping on the Scene. 

* Grab the SpawnPoint Game Objects on the Stage 1 Scene and place them inside the Stage Essential Game Object. 
* Grab the PlayerSpawnPoint Game Object on the Stage 1 Scene and place it inside the Stage Essential Game Object. 
* Grab the Canvas Game Object on the Stage 1 Scene and place it inside the Stage Essential Game Object. 
* Move the SpawnPoints to the rightful location as per the Battle City Game. At the top left, top center and top right of the gameplay area.
* Move the PlayerSpawnPoint to the left of the leftmost brick surrounding the eagle.
* Now your scene should only have 2 game objects at its root. MasterTracker and Stage Essential(everything else is children to Stage Essential). Prefab the MasterTracker and Stage Essential replacing the old ones you have prefab previously.
<img src="{{ site.baseurl }}/images/BattleCity_StageClear_GameOver_3.png" alt="">

Now you are able to create your own stage by dragging the prefab of Stage Essential and MasterTracker to a new scene! Try create a <keyword>Stage2</keyword> Scene by painting a new stage. [Next post]({{ site.baseurl }}{% post_url 2018-03-19-battle-city-unity-16 %}), we are going to talk about creating the electricity animation(Temporary Invincibility) for the Player Tank.