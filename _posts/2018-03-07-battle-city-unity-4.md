---
layout: post
title: "Recreate Battle City in Unity Part 3: Creation of the Protagonist and Antagonists(Tanks)"
excerpt: Recreate Battle City in Unity Part 3
summary: This post is part 3 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

Now we are in the most popular section(but not necessary most important) of any game development, the character creation. Probably due to the abundance of character sprites which looks very attractive, people seem to spend a lot of time on character creation and its animation. 

For me, animations are nice to have and provide an excellent visual experience for players but not necessary something we need to focus much on until we sort out the gameplay itself. In fact, my tanks are crap derivatives to the artists who provided the artwork so pardon my graphics because my artwork sucks. 

How I usually go about animation is by manipulating its transform position, scale and rotation or by looping using two sprites which have minimal differences to create an illusion of an animation.

### Who's your daddy!
Let's start by creating the smallest tank in the game. Drag and drop one of the sprites of your small tank to the Hierarchy and Unity will generate the Game Object for you. You will realize it might not be facing the direction you want.

<div class="info">You will want your enemy tank to be facing down and your player tank to be facing up like in the Battle City Game.</div>

<img src="{{ site.baseurl }}/images/BattleCity_CreatingTanks_1.png" alt="">

You could change the rotation of the Tank Game Object you create by changing it from the transform. But this will make things look a bit messy as the optimal is to have your Game Object Transform Rotation at 0 for X, Y, Z. 

The way to overcome it is to create an empty Game Object and rename the empty Game Object as <keyword>SmallTank</keyword>. Place your Tank Game Object (which you created earlier via drag and drop of your sprite) as a child to the SmallTank Game Object. <keyword>Reset the transform</keyword> of the child and rename the child game object as <keyword>Body</keyword>. Now you can manipulate the rotation in the Body Game Object as we will only be modifying anything to the Tank via the Parent.

<img src="{{ site.baseurl }}/images/BattleCity_CreatingTanks_2.gif" alt="">

<div class="info">If there is a need to rotate the tank, we will rotate it via the Parent, and the child will rotate as well as its transform properties are offset to its parent's. </div>

>Optional: If you wish to create animation for your tank. Check out [my post]({{ site.baseurl }}{% post_url 2018-03-07-unity-2sprites-reusable-animation %}) for that, it teaches you how to create reusable animation with just 2 sprites!

### Rest of the Tanks

Regardless if you intend to create animation or not, it will be straightforward to create the rest of the tanks. Just duplicate the SmallTank and rename the duplicate accordingly to the names you want for the rest of the tanks(mine are FastTank, BigTank and ArmoredTank). Change the sprite(s) in the child Game Object(s) to the different Sprites you have created for the other tanks, and you will have 4 tanks!

<img src="{{ site.baseurl }}/images/BattleCity_TankMoving2.gif" alt="">

That marks the end of creation of the tanks. In the [next post]({{ site.baseurl }}{% post_url 2018-03-08-battle-city-unity-5 %}) we will talk about how to move the tank.