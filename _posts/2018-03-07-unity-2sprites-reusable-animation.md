---
layout: post
title: "How to create reusable 2D animation in Unity with just 2 sprites"
excerpt: How to create reusable animation in Unity with just 2 sprites
summary: This post is part 3 to the series of post detailing how I recreate Battle City in Unity
categories: [How To]
tags: [Unity, Unity 3D, animation, 2D, sprite]
comments: true
---

### Introduction

This post is a spin-off from my tutorial series Recreate Battle City in Unity. A lot of people(me included) find it a chore to create simple 2D animation in Unity as the Animator/Animation/Animator Controller combination is too complex for those.

Like for my case, my animation consists of only 2 sprites, but I need to create it for 4 sets of different sprites. The known way is to use an Animator Override Controller which allows you to reuse the Animator Controller(which sets the States and transitions of animation) but you need to recreate your animation. But what if all the animation is the same only the sprite is different? Why do we need to recreate the animation again(Touching the timeline is a very tedious thing to me)

What I will show here is how we can create a reusable animation(no need for an Animator Override Controller or Animation again) for animation involving two sprites(or more if you adapt accordingly)

### Preparation

The animation I intend to create is tank movement. So all I need to do is to animate its track movement. My two sprites only have a slight derivative at the track itself to create an illusion of moving animation. See below to see what I mean.

<img src="{{ site.baseurl }}/images/BattleCity_TankComparison1.png" alt="">

1. Create an Empty Game Object and rename it accordingly(Mine is called SmallTank). 
2. Drag and drop your 1st sprite to the Hierarchy and Unity will generate the Game Object for you. Rename the Game Object created from your 1st sprite as Body1.
3. Drag and drop your 2nd sprite to the Hierarchy and Unity will generate the Game Object for you. Rename the Game Object created from your 1st sprite as Body2.
4. Drag and drop Body1 and Body2 into SmallTank. This act makes SmallTank the parent Game Object to Body1 and Body2.
5. Reset the transform of Body1 and Body2 so that they are at the same position of the SmallTank Game Object.
6. Disable Body2 Game Object.

<img src="{{ site.baseurl }}/images/Unity_2sprites_reusable_animation_1.gif" alt="">

### One Time Creation of Animator/Animator Controller/Animation
Now we are ready to create the animation of the tank moving. 
1. Go to your Project Window, create an Animation Controller and rename it accordingly(Mine is called Tank). Move the Animation Controller to your preferred folder for organization. 
2. Select the SmallTank Game Object from the hierarchy and add an Animator Component (Miscellaneous -> Animator). 
3. From the Animator Component, set the Controller to point to the Tank Animator Controller you created earlier.

<img src="{{ site.baseurl }}/images/Unity_2sprites_reusable_animation_2.gif" alt="">

Open the Animation Window via the Unity Editor Menu(Window -> Animation). Ensure your SmallTank Game Object is selected in the Hierarchy, click on "Create" at the Animation Window. Unity will prompt for a name for the animation, name it accordingly(Mine is called TankMoving). 

<img src="{{ site.baseurl }}/images/Unity_2sprites_reusable_animation_3.gif" alt="">

Now you should be able to Add Property from the Animation Window. Add two properties(Is Active for Body1 and Is Active for Body2) from the Animation Window.

<img src="{{ site.baseurl }}/images/BattleCity_Animation1.png" alt="">

Once added, you should be able to see something like the below.

<img src="{{ site.baseurl }}/images/BattleCity_Animation2.png" alt="">

Now click on the red circle(symbolizing record). If it is depressed, means we are in recording mode. Click on the timeline at point 0:15, you should have a white line that will move to that position.

<img src="{{ site.baseurl }}/images/BattleCity_Animation3.png" alt="">

Once the white line is at point 0:15, uncheck the box for "Body1: Game Object.Is Active" and check the box for "Body2: Game Object.Is Active".

<img src="{{ site.baseurl }}/images/BattleCity_Animation4.png" alt="">

Now Click on the timeline at point 0:30, that white line should move to that position together.

<img src="{{ site.baseurl }}/images/BattleCity_Animation5.png" alt="">

Once the white line is at point 0:30, check the box for "Body1: Game Object.Is Active" and uncheck the box for "Body2: Game Object.Is Active".

<img src="{{ site.baseurl }}/images/BattleCity_Animation6.png" alt="">

Right-click on one of the rhombus at the point 1:00 and click Delete Key. We do not need that.

<img src="{{ site.baseurl }}/images/BattleCity_Animation7.png" alt="">

Click on the red circle symbolizing record to stop recording. Now your tank moving animation is completed. Click on Play of the Animation to enjoy watching your animation unfold!

<img src="{{ site.baseurl }}/images/BattleCity_TankMoving1.gif" alt="">

### How to reuse this

Reusing this will be straightforward. Create a prefab with the Game Object you just created. Use the prefab to spawn new Game Objects. Change the sprites in the child Game Object Body1 and Body2 to the different Sprites you have created for the duplicates, and you will have a reusable animation for your different sprites. No more Animator Override Controller and Animations!

<img src="{{ site.baseurl }}/images/BattleCity_TankMoving2.gif" alt="">

That marks the end of my post. Hope it helps you in your simple 2D animation!
