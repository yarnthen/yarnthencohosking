---
layout: post
title: "Recreate Battle City in Unity Part 18 : Battle Status Board"
excerpt: Recreate Battle City in Unity Part 18
summary: This post is part 18 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

This post is to design the Battle Status Board(BSB) which is the information panel to right of the Gameplay area. The code itself is not a lot, but creating the design for it is(at least for me). 

### Mimicing Battle City's Battle Status Board
Let's look at the version of the board in Battle City itself.

<img src="{{ site.baseurl }}/images/BattleCity_BSB_1.png" alt="">

We can see the information avaiable are the number of enemy tanks in reserve, stage number and number of player lifes left. The information is fed to the board when 
1. The scene first starts 
2. An enemy tank gets spawned on the gameplay area
3. The player tank gets destroyed

Let's look at my version of BSB.
<img src="{{ site.baseurl }}/images/BattleCity_BSB_2.png" alt="">

I divide the BSB into 3 panels:
* <keyword>EnemyReservePanel</keyword> which is parent to 20 Images of a black tank symbol, the order of the images is from left to right, top to bottom. The images are set to disabled as default.
* <keyword>PlayerLivesPanel</keyword> which is parent to an image of a yellow tank symbol and a text object called <keyword>PlayerLivesText</keyword> which shows the value of how many lives the player has left.
* <keyword>StageNumberPanel</keyword> which is parent to an image of an orange flag and a text object which shows the stage number called <keyword>StageText</keyword>.

### Coding the update of BSB
We will be adding the code for updating the BSB in the <keyword>GamePlayManager</keyword>. Declare a <keyword>Transform</keyword> Variable named <keyword>tankReservePanel</keyword> and 2 <keyword>Text</keyword> Variables <keyword>playerLivesText</keyword> and <keyword>stageNumber</keyword>, all with <keyword>SerializeField</keyword>. Drag and drop the EnemyReservePanel game object to tankReservePanel, PlayerLivesText game object to playerLivesText and StageText game object to stageNumber in the Inspector of Canvas's component GamePlayManager Script. Also declare a <keyword>GameObject</keyword> variable called <keyword>tankImage</keyword> which will be used to store reference to the tank image in the EnemyReservePanel.

{% highlight csharp%}
public class GamePlayManager : MonoBehaviour {
    [SerializeField]
    Transform tankReservePanel;
    [SerializeField]
    Text playerLivesText, stageNumber;
    GameObject tankImage;
   //rest of the earlier code are omitted for focus
{% endhighlight %}
<img src="{{ site.baseurl }}/images/BattleCity_BSB_3.png" alt="">

### Update the Enemy Reserve Panel

We will need 2 routines for this. One called <keyword>UpdateTankReserve</keyword> which is to update the panel at the start of the stage, the next one called <keyword>RemoveTankReserve</keyword> will be a Public routine which will be called by the Spawner script's StartSpawning routine to update the panel when a new enemy tank is spawned.
{% highlight csharp%}
void UpdateTankReserve()
{
    int j;
    int numberOfTanks = LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks;
    for (j = 0; j < numberOfTanks; j++)
    {
        tankImage = tankReservePanel.transform.GetChild(j).gameObject;
        tankImage.SetActive(true);
    }
}
public void RemoveTankReserve()
{
    int numberOfTanks = LevelManager.smallTanks + LevelManager.fastTanks + LevelManager.bigTanks + LevelManager.armoredTanks;
    tankImage = tankReservePanel.transform.GetChild(numberOfTanks).gameObject;
    tankImage.SetActive(false);
}
{% endhighlight %}

<div class="info">UpdateTankReserve gets the number of tanks in reserve from Level Manager. Based on that figure, set active the same number of tank images(the tank images are set inactive by default). There is a maximum of 20 images which is the number of tanks that Battle City has per stage. RemoveTankReserve get the number of tanks left in reserve from Level Manager. Based on that figure, set to inactive the tank image that is at position same as the number of tanks left in reserve.</div>

Then update the RemoveTankReserve routine to the StartSpawning routine in Spawner script. The full code for the routine as below.

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
        tank.transform.SetParent(enemyHolder);
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

### Update the Player Lives Panel

The code for this is very simple, get the value of the player lives left from MasterTracker to the playerLivesText game object. The routine to hold this code will be called <keyword>UpdatePlayerLives</keyword>.
{% highlight csharp%}
public void UpdatePlayerLives()
{
    playerLivesText.text = MasterTracker.playerLives.ToString();
}
{% endhighlight %}

### Update the Stage Number Panel
The code for this is very simple, get the value of the Stage Number from MasterTracker to the StageText game object. The routine to hold this code will be called <keyword>UpdateStageNumber</keyword>.
{% highlight csharp%}
void UpdateStageNumber()
{
    stageNumber.text = MasterTracker.stageNumber.ToString();
}
{% endhighlight %}

### Putting everything together

Now we can add the routines <keyword>UpdateTankReserve, UpdatePlayerLives and UpdateStageNumber</keyword> to the <keyword>Start</keyword> Monobehaviour of <keyword>GamePlayManager</keyword>. Full code for the Start Monobehaviour as below.
{% highlight csharp%}
void Start () {
    stageNumberText.text = "STAGE " + MasterTracker.stageNumber.ToString();
    spawnPoints = GameObject.FindGameObjectsWithTag("EnemySpawnPoint");
    spawnPlayerPoints = GameObject.FindGameObjectsWithTag("PlayerSpawnPoint");
    steelTilemap = GameObject.Find("Steel").GetComponent<Tilemap>();
    waterTilemap = GameObject.Find("Water").GetComponent<Tilemap>();
    UpdateTankReserve();
    UpdatePlayerLives();
    UpdateStageNumber();
    StartCoroutine(StartStage());
}
{% endhighlight %}

Let's try out the game and see that it gets updated. I will first start the game by setting the Stage to 1 and the number of enemy tanks to 20. Then I will stop the game and update the Stage to 2 and the number of tanks to 8. Later I will delibrate get my tank destroyed to see if all the updates are working.
<img src="{{ site.baseurl }}/images/BattleCity_BSB_4.gif" alt="">

Well, it took a while before the enemy decide to destroy me. But everything tested successfully! 

Next up will be on the Bonus crates creation.