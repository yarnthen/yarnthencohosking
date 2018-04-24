---
layout: post
title: "Recreate Battle City in Unity Part 17 : Score Calculation"
excerpt: Recreate Battle City in Unity Part 17
summary: This post is part 15 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

I would like you to go back and create the other 3 enemy tanks using the EnemyTank prefab you already have. You can do a recap on [Part 3]({{ site.baseurl }}{% post_url 2018-03-07-battle-city-unity-4 %}) if you are unsure. Set the following below for the tanks as well

| Prefab Name | Tag |
|:-:|:-:|
|SmallTank|Small|
|FastTank|Fast|
|BigTank|Big|
|ArmoredTank|Armored|

At the end you should have 4 enemy tanks prefabs like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_1.png" alt="">

Then update the SpawnPoints's Spawner (Script) component with the updated prefabs.

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_2.png" alt="">

### How the Score Calculation looks

Let's take a look at the Score Calculation Scene from Battle City

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_3.png" alt="">

This is the scene that will show the score. You can see that all the information it requires can be taken from the <keyword>MasterTracker</keyword> so there should not be many issues. The problem lies in replication of the scene by me(the artless person). Let's try it out. Start by creating a new scene and name it as <keyword>Score</keyword>. 

First create an empty Game Object (call it <keyword>ScoreScene</keyword>)on the screen and reset its transform. Create an image as a child of ScoreScene which will, in turn, create a canvas and EventSystem as a child of ScoreScene. The image will be a child of the canvas. Call the Image BlackScreen. Set its Image Color to Black. Click on the "middle/center" graph in the Inspector Rect Transform and then, while holding Alt key, press on "stretch/stretch". This will allow the BlackScreen to cover the entire canvas.

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_4.gif" alt="">

The rest of the UI items are created inside 3 panels inside the Canvas. Below are the settings I made to create the effect similar to Battle 
City. <keyword>After making the settings below, you need to set the Left, Top, Right, Bottom to 0 for the positions to move.</keyword>

|Name|MinX|MinY|MaxX|MaxY|Comments|
|:---|:-:|:-:|:-:|:-:|---|
|Panel1|0|0.8|1|1|Child to the Canvas.<br> Change the Panel Color alpha to 0|
|Panel2|0|0.6|1|0.8|Child to the Canvas.<br> Change the Panel Color alpha to 0|
|Panel3|0|0|1|0.6|Child to the Canvas.<br> Change the Panel Color alpha to 0|
|TextHiScore|0|0.5|0.5|1|Child to the Panel1. <br>Display the HI-SCORE word|
|HiScoreText|0.5|0.5|1|1|Child to the Panel1. <br>Text to display will be passed by code|
|StageText|0|0|1|0.5|Child to the Panel1. <br>Text to display will be passed by code|
|PlayerText|0|0.5|0.3|1|Child to the Panel2.<br>Display the I-PLAYER word|
|PlayerScoreText|0|0|0.3|0.5|Child to the Panel2. <br>Text to display will be passed by code|
|PtsText1|0.2|0.8|0.3|1|Child to the Panel3. <br>Display the PTS word|
|PtsText2|0.2|0.6|0.3|0.8|Child to the Panel3. <br>Display the PTS word|
|PtsText3|0.2|0.4|0.3|0.6|Child to the Panel3. <br>Display the PTS word|
|PtsText4|0.2|0.2|0.3|0.4|Child to the Panel3. <br>Display the PTS word|
|TotalText|0|0|0.3|0.2|Child to the Panel3. <br>Display the TOTAL word|
|SmallTankScore|0|0.8|0.2|1|Child to the Panel3. <br>Text to display will be passed by code|
|FastTankScore|0|0.6|0.2|0.8|Child to the Panel3. <br>Text to display will be passed by code|
|BigTankScore|0|0.4|0.2|0.6|Child to the Panel3. <br>Text to display will be passed by code|
|ArmoredTankScore|0|0.2|0.2|0.4|Child to the Panel3. <br>Text to display will be passed by code|
|SmallTanksDestroyed|0.3|0.8|0.4|1|Child to the Panel3. <br>Text to display will be passed by code|
|FastTanksDestroyed|0.3|0.6|0.4|0.8|Child to the Panel3. <br>Text to display will be passed by code|
|BigTanksDestroyed|0.3|0.4|0.4|0.6|Child to the Panel3. <br>Text to display will be passed by code|
|ArmoredTanksDestroyed|0.3|0.2|0.4|0.4|Child to the Panel3. <br>Text to display will be passed by code|
|TotalTanksDestroyed|0.3|0|0.4|0.2|Child to the Panel3. <br>Text to display will be passed by code|
|Arrow1|0.4|0.85|0.45|0.95|Child to the Panel3. <br>Display the arrow symbol|
|Arrow2|0.4|0.65|0.45|0.75|Child to the Panel3. <br>Display the arrow symbol|
|Arrow3|0.4|0.45|0.45|0.55|Child to the Panel3. <br>Display the arrow symbol|
|Arrow4|0.4|0.25|0.45|0.35|Child to the Panel3. <br>Display the arrow symbol|
|SmallTankImage|0.48|0.86|0.52|0.94|Child to the Panel3. <br>For display the tank image.<br> Values are adjusted to fit my images so it might not fit yours|
|FastTankImage|0.48|0.66|0.52|0.74|Child to the Panel3. <br>For display the tank image.<br> Values are adjusted to fit my images so it might not fit yours|
|BigTankImage|0.48|0.45|0.52|0.55|Child to the Panel3. <br>For display the tank image.<br> Values are adjusted to fit my images so it might not fit yours|
|ArmoredTankImage|0.48|0.24|0.52|0.36|Child to the Panel3. <br>For display the tank image.<br> Values are adjusted to fit my images so it might not fit yours|

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_5.png" alt="">

### Coding the calculation of Score

Now we are ready to start coding. Create a script called TallyScore on the Canvas. First thing we need to do on the script is to add <keyword>using UnityEngine.UI</keyword> in order to use features related to UI and <keyword>using UnityEngine.SceneManagement</keyword> to use its static routine to move to next scene. 
* Declare the follwing <keyword>Text</keyword> variables with <keyword>SerializeField</keyword>.
hiScoreText, stageText, playerScoreText, smallTankScoreText, fastTankScoreText, bigTankScoreText, armoredTankScoreText, smallTanksDestroyed, fastTanksDestroyed, bigTanksDestroyed, armoredTanksDestroyed, totalTanksDestroyed. These are to store references to the Text elements in the UI so that we can update them.
* Declare 4 <keyword>integer</keyword> variables smallTankScore, fastTankScore, bigTankScore, armoredTankScore. This is to help store values during the calculation of points
* Declare 1 <keyword>MasterTracker</keyword> variable called masterTracker to store the reference to MasterTracker to access its public variables.
* Declare 4 <keyword>integer</keyword> variables smallTankPointsWorth, fastTankPointsWorth, bigTankPointsWorth, armoredTankPointsWorth to store the values of MasterTracker public variables we need.


At the Start MonoBehaviour, update to the below for the routine.

{% highlight csharp%}
void Start () {
    masterTracker = GameObject.Find("MasterTracker").GetComponent<MasterTracker>();
    smallTankPointsWorth = masterTracker.smallTankPointsWorth;
    fastTankPointsWorth = masterTracker.fastTankPointsWorth;
    bigTankPointsWorth = masterTracker.bigTankPointsWorth;
    armoredTankPointsWorth = masterTracker.armoredTankPointsWorth;
    stageText.text = "STAGE " + MasterTracker.stageNumber;
    playerScoreText.text = MasterTracker.playerScore.ToString();
}


{% endhighlight %}

<div class="info"> The above is to get the reference of MasterTracker to the variable masterTracker and use that reference to get the public variables to save to smallTankScore, fastTankScore, bigTankScore and armoredTankScore. And update the stageText with the correct Stage number and the Player Score before the calculation we are doing</div>

Create a coroutine called <keyword>UpdateTankPoints</keyword> to do the update. The first part of this coroutine is to loop the number of tanks destroyed one by one and multiply with how much points each tank is worth. This gives the effect that Battle City is doing, counting 1 by 1. Then we will show the total number of tanks destroyed followed by adding it to the PlayerScore in MasterTracker.
{% highlight csharp%}
IEnumerator UpdateTankPoints()
{
    for (int i = 0; i <= MasterTracker.smallTanksDestroyed; i++)
    {
        smallTankScore = smallTankPointsWorth * i;
        smallTankScoreText.text = smallTankScore.ToString();
        smallTanksDestroyed.text = i.ToString();
        yield return new WaitForSeconds(0.2f);
    }
    for (int i = 0; i <= MasterTracker.fastTanksDestroyed; i++)
    {
        fastTankScore = fastTankPointsWorth * i;
        fastTankScoreText.text = fastTankScore.ToString();
        fastTanksDestroyed.text = i.ToString();
        yield return new WaitForSeconds(0.2f);
    }
    for (int i = 0; i <= MasterTracker.bigTanksDestroyed; i++)
    {
        bigTankScore = bigTankPointsWorth * i;
        bigTankScoreText.text = bigTankScore.ToString();
        bigTanksDestroyed.text = i.ToString();
        yield return new WaitForSeconds(0.2f);
    }
    for (int i = 0; i <= MasterTracker.armoredTanksDestroyed; i++)
    {
        armoredTankScore = armoredTankPointsWorth * i;
        armoredTankScoreText.text = armoredTankScore.ToString();
        armoredTanksDestroyed.text = i.ToString();
        yield return new WaitForSeconds(0.2f);
    }
    totalTanksDestroyed.text = (MasterTracker.smallTanksDestroyed + MasterTracker.fastTanksDestroyed + MasterTracker.bigTanksDestroyed + MasterTracker.armoredTanksDestroyed).ToString();
    MasterTracker.playerScore += (smallTankScore + fastTankScore + bigTankScore + armoredTankScore);
}
{% endhighlight %}

At the end part of the coroutine, . Then we will add a delay for 5 seconds for the player to absorb their achievements before going to the next scene depending on whether the player has cleared the stage(stageCleared = true) or game over(stageCleared = false). Before going to the next scene, we will trigger another routine we will create (called ClearStatistics) to reset values in MasterTracker which will be reused again.

{% highlight csharp%}
//continuation of UpdateTankPoints coroutine
    yield return new WaitForSeconds(5f);
    if (MasterTracker.stageCleared)
    {
        ClearStatistics();
        SceneManager.LoadScene("Stage" + (MasterTracker.stageNumber + 1));
    }
    else
    {
        ClearStatistics();
        //Load GameOver Scene
    }
}
void ClearStatistics()
{
    MasterTracker.smallTanksDestroyed = 0;
    MasterTracker.fastTanksDestroyed = 0;
    MasterTracker.bigTanksDestroyed = 0;
    MasterTracker.armoredTanksDestroyed = 0;
    MasterTracker.stageCleared = false;
}
{% endhighlight %}

Then finally we can trigger the coroutine by the Start Monobehaviour. Add StartCoroutine(UpdateTankPoints()) to the last part of the routine. The full code for TallyScore as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class TallyScore : MonoBehaviour {
    [SerializeField]
    Text hiScoreText, stageText, playerScoreText, smallTankScoreText, fastTankScoreText, bigTankScoreText, armoredTankScoreText, smallTanksDestroyed, fastTanksDestroyed, bigTanksDestroyed, armoredTanksDestroyed, totalTanksDestroyed;
    int smallTankScore, fastTankScore, bigTankScore, armoredTankScore;
    MasterTracker masterTracker;
    int smallTankPointsWorth, fastTankPointsWorth, bigTankPointsWorth, armoredTankPointsWorth;
    // Use this for initialization
    void Start () {
        masterTracker = GameObject.Find("MasterTracker").GetComponent<MasterTracker>();
        smallTankPointsWorth = masterTracker.smallTankPointsWorth;
        fastTankPointsWorth = masterTracker.fastTankPointsWorth;
        bigTankPointsWorth = masterTracker.bigTankPointsWorth;
        armoredTankPointsWorth = masterTracker.armoredTankPointsWorth;
        stageText.text = "STAGE " + MasterTracker.stageNumber;
        playerScoreText.text = MasterTracker.playerScore.ToString();
        StartCoroutine(UpdateTankPoints());
    }
    IEnumerator UpdateTankPoints()
    {
        for (int i = 0; i <= MasterTracker.smallTanksDestroyed; i++)
        {
            smallTankScore = smallTankPointsWorth * i;
            smallTankScoreText.text = smallTankScore.ToString();
            smallTanksDestroyed.text = i.ToString();
            yield return new WaitForSeconds(0.2f);
        }
        for (int i = 0; i <= MasterTracker.fastTanksDestroyed; i++)
        {
            fastTankScore = fastTankPointsWorth * i;
            fastTankScoreText.text = fastTankScore.ToString();
            fastTanksDestroyed.text = i.ToString();
            yield return new WaitForSeconds(0.2f);
        }
        for (int i = 0; i <= MasterTracker.bigTanksDestroyed; i++)
        {
            bigTankScore = bigTankPointsWorth * i;
            bigTankScoreText.text = bigTankScore.ToString();
            bigTanksDestroyed.text = i.ToString();
            yield return new WaitForSeconds(0.2f);
        }
        for (int i = 0; i <= MasterTracker.armoredTanksDestroyed; i++)
        {
            armoredTankScore = armoredTankPointsWorth * i;
            armoredTankScoreText.text = armoredTankScore.ToString();
            armoredTanksDestroyed.text = i.ToString();
            yield return new WaitForSeconds(0.2f);
        }
        totalTanksDestroyed.text = (MasterTracker.smallTanksDestroyed + MasterTracker.fastTanksDestroyed + MasterTracker.bigTanksDestroyed + MasterTracker.armoredTanksDestroyed).ToString();
        MasterTracker.playerScore += (smallTankScore + fastTankScore + bigTankScore + armoredTankScore);
        yield return new WaitForSeconds(5f);
        if (MasterTracker.stageCleared)
        {
            ClearStatistics();
            SceneManager.LoadScene("Stage" + (MasterTracker.stageNumber + 1));
        }
        else
        {
            ClearStatistics();
            //Load GameOver Scene
        }
    }
    void ClearStatistics()
    {
        MasterTracker.smallTanksDestroyed = 0;
        MasterTracker.fastTanksDestroyed = 0;
        MasterTracker.bigTanksDestroyed = 0;
        MasterTracker.armoredTanksDestroyed = 0;
        MasterTracker.stageCleared = false;
    }
}
{% endhighlight %}

Drag and drop the right text component to its placeholder in Canvas's Tally Score (Script) component.

<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_6.gif" alt="">

Prefab the ScoreScene Game Object so that all its children will be part of the prefab and save the Score scene.
Now we are ready to test. If you have yet to create a <keyword>Stage2</keyword> Scene, please do so now. Once it is done, go to Unity's File->Build Settings and add Stage1, Score and Stage 2 to it. Remember to update the Level Manager for the Stage Number in Stage2.
<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_7.gif" alt="">

Now go back to Stage 1 and start the game!
<img src="{{ site.baseurl }}/images/BattleCity_Score_Calculation_8.gif" alt="">


### Time to design stages

Yeah! Its working and it will be very easy to create new stages.  Next post will be the last and it's on the Battle Status Board.


