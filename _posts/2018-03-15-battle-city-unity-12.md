---
layout: post
title: "Recreate Battle City in Unity Part 11 : The Gamemaker - Starting the Stage and Game Over"
excerpt: Recreate Battle City in Unity Part 11
summary: This post is part 11 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

### Introduction

The Gamemaker controls the flow of the entire gameplay:
1. When and how to start the stage
2. When and how to end the stage(Gameover or stage completion)
3. When and how to spawn objects(Player, enemies and bonus crates)

We will start with the first part (When and how to start the stage).

### Starting the Game Stage
Let's start by looking at the Start Stage Animation.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_1.gif" alt="">
We can see the grey curtains pulling in from the top and bottom to the center to review the Stage Number. Then the grey curtain is pulling out from the center to top and bottom to reveal the stage. To create this effect, we will need 2 images for the curtain and a text field for the Stage Number Text. Create an image(<keyword>UI->Image</keyword>) from the hierarchy. 

<div class="info">Unity will create 2 Game Objects, <keyword>Canvas</keyword> and <keyword>EventSystem</keyword>. We will only be using the Canvas for this section.</div> 

The image will be created as a child of the Canvas. Rename the image as <keyword>TopCurtain</keyword>. Create another image by right click on the Canvas (<keyword>UI->Image</keyword>). Rename the new image <keyword>BottomCurtain</keyword>. Reset the transform of the two images. 

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_2.png" alt="">
 
* Select the TopCurtain. Set Anchors Min X to 0, Min Y to 0.5, Max X to 1 and Max Y to 1 in the Inspector.
* Set the Rect Transform->Left, Top, Right, and Bottom to 0.
<div class="info"> This is to make sure the image takes up the top half of the canvas regardless any changes to the screen size.</div>
* Select the BottomCurtain. Set Anchors Min X to 0, Minx Y to 0, Max X to 1 and Max Y to 0.5 in the Inspector.
* Set the Rect Transform->Left, Top, Right, and Bottom to 0.
<div class="info">This is to make sure the image takes up the bottom half of the canvas regardless any changes to the screen size.</div>

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_3.gif" alt="">

* Create a Text UI by right click on the Canvas (<keyword>UI->Text</keyword>). 
* Rename the Text UI as StageNumberText. 
* Set the width of the Rect Transform to 200 and Height to 60.
* Set the Paragraph alignment to center-horizontal and center-vertical.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_4.gif" alt="">

* Create a 3rd image in the Canvas. Rename it as BlackCurtain. 
* Set its Image Color to Black. Click on the "middle/center" graph in the Inspector Rect Transform and then, while holding Alt key, press on "stretch/stretch". This is to create the effect of grey curtains pulling in from the top and bottom to the center to reveal the Stage Number.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_5.gif" alt="">

<div class="info">Notice the canvas in the hierarchy? The sequence of the children(TopCurtain, BottomCurtain, StageNumberText, and lastly BlackCurtain) is deliberate. This sequence will ensure BlackCurtain will render on top of the rest of the items on the canvas.</div>

# Coding the curtain effects

Create a Script Component named GamePlayManager under the Canvas Game Object.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_6.png" alt="">

We need to use the functions available to the UI so we can manipulate the UI Objects we added. Add "<keyword>using UnityEngine.UI;</keyword>" at the top of the script.

Declare 4 serialized field variables:
1. 3 Image variable called <keyword>topCurtain</keyword>, <keyword>bottomCurtain</keyword> and <keyword>blackCurtain</keyword>
2. 1 Text variable called <keyword>stageNumberText</keyword>.

{% highlight csharp%}
[SerializeField]
Image topCurtain, bottomCurtain, blackCurtain;
[SerializeField]
Text stageNumberText;
{% endhighlight %}

Let's start with the revealing of Stage Number effect. All we need to do for this effect is to decrease the scale.y of the BlackCurtain until it reaches 0 so that it will show the Stage Number behind. To do that, we will use a coroutine(named <keyword>RevealStageNumber</keyword>) as a means for creating the gradual decrease in size. Use a Mathf.Clamp to ensure the Scale does not go beyond 0 as a negative value will mean the curtain starts to expand in size again.

{% highlight csharp%}
IEnumerator RevealStageNumber()
{
    while (blackCurtain.rectTransform.localScale.y > 0)
    {
        blackCurtain.rectTransform.localScale = new Vector3(blackCurtain.rectTransform.localScale.x, Mathf.Clamp(blackCurtain.rectTransform.localScale.y - Time.deltaTime,0,1), blackCurtain.rectTransform.localScale.z);
        yield return null;
    }
}
{% endhighlight %}

Now let's test it. Add the StartCoroutine RevealStageNumber to the <keyword>Start</keyword> Monobehaviour routine. 

{% highlight csharp%}
void Start () {
    StartCoroutine(RevealStageNumber());
}
{% endhighlight %}

Drag and drop the BlackCurtain Image Object to the GamePlayManager Component's variable blackCurtain and press on Play.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_7.gif" alt="">

Done. Now let's move on the "grey curtain pulling out from the center to top and bottom to reveal the stage" effect. Let's start by disabling the BlackCurtain Game Object. First, start by creating a <keyword>SerializedField RecTransform</keyword> variable called <keyword>canvas</keyword> to store the Canvas RecTransform property, we are using it to find out the height of the Canvas.

<div class="info">We can't do the same like for the BlackCurtain, as the outcome will not be what we want to achieve.</div> 

We will achieve the effect by pulling the TopCurtain to the top and pulling the BottomCurtain to the bottom. Let's do all these via 2 coroutines called <keyword>RevealTopStage</keyword> and <keyword>RevealBottomStage</keyword>. We will also be disabling the StageNumberText when the coroutines run. The float variables <keyword>moveTopUpMin</keyword> and <keyword>moveBottomDownMin</keyword> are the calculated value of how much the topCurtain and bottomCurtain need to move to be out of the scene.

<div class="info">The moveTopUpMin and moveBottomDownMin are calculated by adding(move up) or subtracting(move down) the position of the curtain by half the height of the canvas plus a buffer of 10</div>

{% highlight csharp%}
[SerializeField]
RectTransform canvas;

IEnumerator RevealTopStage()
{
    float moveTopUpMin = topCurtain.rectTransform.position.y + (canvas.rect.height / 2)  + 10;
    stageNumberText.enabled = false;
    while (topCurtain.rectTransform.position.y < moveTopUpMin)
    {
        topCurtain.rectTransform.Translate(new Vector3(0, 500 * Time.deltaTime, 0));
        yield return null;
    }
}
IEnumerator RevealBottomStage()
{
    float moveBottomDownMin = bottomCurtain.rectTransform.position.y - (canvas.rect.height / 2) - 10 ;
    while (bottomCurtain.rectTransform.position.y > moveBottomDownMin)
    {
        bottomCurtain.rectTransform.Translate(new Vector3(0, -500 * Time.deltaTime, 0));
        yield return null;
    }
}
{% endhighlight %}

Now let's test it. Add the StartCoroutine RevealTopStage and RevealBottomStage to the Start Monobehaviour. We will comment out the RevealStageNumber coroutine.

{% highlight csharp%}
void Start () {
    //StartCoroutine(RevealStageNumber());
    StartCoroutine(RevealTopStage());
    StartCoroutine(RevealBottomStage());
}
{% endhighlight %}

<keyword>Disable the Black Curtain Game Object</keyword>. Drag and drop the TopCurtain, BottomCurtain Image and StageNumberText and Canvas Objects to the GamePlayManager Component's variable topCurtain, bottomCurtain, stageNumberText and canvas respectively and press on Play.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_8.gif" alt="">

Let's merge both via another coroutine to trigger the routine one after another. The routine will be called <keyword>StartStage</keyword>. Change the coroutine to trigger at the Start Monobehaviour to StartScene.

{% highlight csharp%}
void Start () {
        StartCoroutine(StartStage());
}
IEnumerator StartStage()
{
    StartCoroutine(RevealStageNumber());
    yield return new WaitForSeconds(5);
    StartCoroutine(RevealTopStage());
    StartCoroutine(RevealBottomStage());
}
{% endhighlight %}

Now let's test the whole sequence again. Enable back the BlackCurtain if you have disabled it earlier.

<img src="{{ site.baseurl }}/images/BattleCity_TheGameMaker_9.gif" alt="">

That is all for the starting a stage sequence. The full code for <keyword>GamePlayManager</keyword> until now as below.

{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class GamePlayManager : MonoBehaviour {
    [SerializeField]
    Image topCurtain, bottomCurtain, blackCurtain;
    [SerializeField]
    Text stageNumberText;
    // Use this for initialization
    void Start () {
        StartCoroutine(StartStage());
    }
    IEnumerator StartStage()
    {
        StartCoroutine(RevealStageNumber());
        yield return new WaitForSeconds(5);
        StartCoroutine(RevealTopStage());
        StartCoroutine(RevealBottomStage());
    }
	IEnumerator RevealStageNumber()
    {
        while (blackCurtain.rectTransform.localScale.y > 0)
        {
            blackCurtain.rectTransform.localScale = new Vector3(1, Mathf.Clamp(blackCurtain.rectTransform.localScale.y - Time.deltaTime,0,1), 1);
            yield return null;
        }
    }
    IEnumerator RevealTopStage()
    {
        stageNumberText.enabled = false;
        while (topCurtain.rectTransform.position.y < 1250)
        {
            topCurtain.rectTransform.Translate(new Vector3(0, 500 * Time.deltaTime, 0));
            yield return null;
        }
    }
    IEnumerator RevealBottomStage()
    {
        while (bottomCurtain.rectTransform.position.y > -400)
        {
            bottomCurtain.rectTransform.Translate(new Vector3(0, -500 * Time.deltaTime, 0));
            yield return null;
        }
    }
}
{% endhighlight %}

### Game over, play again?

Now, let's take a look at the GameOver scene.

<img src="{{ site.baseurl }}/images/BattleCity_GameOver_1.gif" alt="">

The GameOver portion is quite straightforward. Just a text mentioning Game Over scrolling up. We can do this easily in the GamePlayerManager script. 
* Disable all the UI elements under <keyword>Canvas</keyword> to concentrate just on the Game Over portion                         
* Create a Text UI game object in the Canvas calling it <keyword>GameOverText</keyword>. 
* Set the width of the Rect Transform to 200 and Height to 60.
* Set its Paragraph alignment to center-horizontal and center-vertical.
* Update its Text to GAME OVER.
* Change the color to Red like in the Battle City.

<img src="{{ site.baseurl }}/images/BattleCity_GameOver_2.png" alt="">

Let's move the GameOverText to the below the Canvas so that we cannot see it anymore. We will do it by changing its Rec Transform (Anchor Min Y = 0, Achnor Max Y = 0, Rect Transform PosY = -30) in inspector shown below.

<img src="{{ site.baseurl }}/images/BattleCity_GameOver_3.png" alt="">

### Code the Game Over

The code for Game Over is to move the GameOverText from the bottom to the center of the canvas. The Same thing as the Start Stage, we will be using a coroutine for the gradual move to position. We will need another Text Variable that is SerializeField(called <keyword>gameOverText</keyword>) so that we can manipulate the text's position later.

{% highlight csharp%}
[SerializeField]
Text stageNumberText, gameOverText; // addon to the existing
{% endhighlight %}

Next is the coroutine(called <keyword>GameOver</keyword>) to move it into position. Make the coroutine Public as it is required for use outside as well later.

<div class="info">The value in the code below (120f) is to control the speed of the GameOverText, it is derived via trial and error. It might not suit your taste so change it accordingly.</div>

{% highlight csharp%}
public IEnumerator GameOver()
{
    while (gameOverText.rectTransform.localPosition.y < 0)
    {
        gameOverText.rectTransform.localPosition = new Vector3(gameOverText.rectTransform.localPosition.x, gameOverText.rectTransform.localPosition.y + 120f * Time.deltaTime, gameOverText.rectTransform.localPosition.z);
        yield return null;
    }
}
{% endhighlight %}

Let's test it now. Change the <keyword>Start</keyword> Monobehaviour for <keyword>GamePlayManager</keyword> to the below.

{% highlight csharp%}
void Start () { 
    StartCoroutine(GameOver());
}
{% endhighlight %}

Drag and drop the GameOverText Object to the GamePlayManager Component's variable gameOverText and press on Play.

<img src="{{ site.baseurl }}/images/BattleCity_GameOver_4.gif" alt="">

That's it for the Start Stage and GameOver. [Next stop]({{ site.baseurl }}{% post_url 2018-03-16-battle-city-unity-13 %}), we are going to talk about the LevelManager, which allows us to do some settings on individual stages.
