---
layout: post
title: "Battle City in Unity Part 2: Level Creation using Tilemaps"
excerpt: Battle City in Unity Part 2
summary: This post is part 2 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
series: "Recreate Battle City in Unity"
---
{% include serieswithintro.html %}

<div class="info">If you have not save your scene, now is a good time to do it.</div>

### One Game Object to rule them all

Create an empty Game Object and name it <keyword>Stage Essential</keyword>(or whatever name you want). Stage Essential will remain an empty Game Object as its sole purpose is to act as a <keyword>parent to all the rest of the Game Objects</keyword> we are going to create outside of the Master Tracker. 

<div class="info">Master Tracker needs to remain at the root of the hierarchy to achieve its singleton pattern effect.</div>

Next, <keyword>reset the transform of Stage Essential</keyword>. This is VERY IMPORTANT not optional. 

<div class="info">We are using Stage Essential to be the parent of all the other Game Objects, once a Game Object is "child", its transform position will not be showing as per what is the Scene Position instead it will be relative to its parent. So by keeping the parent at the origin, the transform position of child Game Object will be the same as its Scene Position.</div>

Move the <keyword>Main Camera</keyword> to the Stage Essential. Your Hierarchy with the Stage Essential Inspector should like the below.

<img src="{{ site.baseurl }}/images/BattleCity_UnityHierarchy2.png" alt="">

### Tilemap Preparation

The main reason I choose Battle City as my first retro game to recreate is because of the new Tilemap feature which was only introduced in Unity in late 2017. As it is still new, there is still lack of information on it so using it allows me to familiarize with it as I believe it is a very powerful tool to aid in 2D Game development. If you are unfamiliar with it, I suggest you to go through the [tutorials from Unity](https://unity3d.com/learn/tutorials/topics/2d-game-creation/intro-2d-world-building-w-tilemap?playlist=17093) for a start.

1. Create a tilemap in the hierarchy from <keyword>2D Objects -> Tilemap</keyword>. This will generate a Game Object called <keyword>Grid</keyword>, and the <keyword>Tilemap</keyword> will be a child to the Grid. The purpose of the Grid other than being a parent is acting as guides with gridlines of where the tiles will be painted. 
2. Move the Grid and its child into Stage Essential. 
3. Rename the Tilemap as <keyword>Ground</keyword>; this will be used to place the tiles for the ground of the stage. 
4. Create 6 more tilemaps renaming them as <keyword>Boundary, Bricks, Steel, Canopy, Water, and TilePalette</keyword>. 

<div class="info">Each of these tilemaps will represent the characteristics of each tile type(e.g., what it can collide with and what happens to it when collided), so creating multiple tilemaps allows the flexibility of creating individual characteristics later.</div>

At this point, your Hierarchy should look like the below.

<img src="{{ site.baseurl }}/images/BattleCity_UnityHierarchy3.png" alt="">

Once the above is done, go to the individual tilemap you have created and change the <keyword>Tilemap Renderer -> Order in Layer</keyword> and the <keyword>Layer</keyword> and the <keyword>Tag</keyword> in the Inspector. The places to update are highlighted below.

<img src="{{ site.baseurl }}/images/BattleCity_UnityHierarchy4.png" alt="">

<div class="info"><keyword>Order in Layer</keyword> setting is to tell which Tilemap will be rendered on top of the other based on their Sorting Layer. We will be only using the Default Sorting Layer, so we just need to change the Order in Layer. A tile in a tilemap that is in Order in Layer 1 will be rendered below that of a tile of Order in Layer 2 if they are placed in the same position.<br> 
<keyword>Layer</keyword> is used here for setting which of the layers can collide with another via the physics 2D settings.<br> 
<keyword>Tag</keyword> is to give a means to identify the Tiles in the Tilemap later via code.</div> 

Set Order in Layer, Layer, and tag as below. If the layer or tag is not available for selection in the drop-down, you will need to create it.

| Tilemap Name | Order In Layer | Tag | Layer |
|-|:-:|:-:|:-:|
| Ground | -1 | Untagged | Default | 
| Bricks | 0 <br>(default)| Brick | Brick |
| Steel | 0 <br>(default)| Steel | Boundary |
| Water | 0 <br>(default)|  Untagged |Water |
| Canopy | 1 | Untagged | Default |
| Boundary | 0 <br>(default)| Untagged | Boundary |
| TilePalette | 0 <br>(default)| Untagged | Default |

<div class="info">Notice I set the Ground Tilemap with Order in Layer -1? This is for convenience sake later as all GameObject created with Sprite Renderer has its Order in Layer defaulted to 0. Then you do not need to adjust any of the new Game Objects Sprite Renderer Order in Layer as it will default to be 0 which is above the Ground Tilemap's -1 thus rendering the GameObject above the Ground.</div>

### Tile Palette Preparation

The next step is prepare the Tile Palette. a Tile Palette is a place where you gather different tiles to be painted on the Tilemap. To get to the Tile Palette, you will need to select <keyword>Window->Tile Palette</keyword> from the Unity Editor. Start by clicking Create New Palette in the Tile Palette window and name it Palette. Some basic orientation on the Tile Palette below.


<b>Purple highlight:</b> The Palette that is in use. I have seen people using different palettes for organization purposes. For this project, there are only limited type of tiles in use so I will just stick to one Palette.
<br>
<b>Yellow highlight:</b> The Active Tilemap. This is probably the most important thing to pay attention to. For example if I paint a brick tile with the Steel Tilemap selected. It would result in a brick tile acting like a steel tile(remember the tilemaps are assigned different layers which I will be changing the Physics 2D setting to choose what they can collide with). Imagine you have painted for 10 minutes and you realize you used the wrong tilemap!
<br>
<b>Pink highlight:</b> Tools for actions that you can do on a tilemap or tile palette. These are generic paint actions of Select, Move, Paint, Fill, Color Picker, Eraser, Paint Large Area.
<br>
<b>Green highlight:</b> Where you will perform the actions of the tools. If the Edit is depressed, your actions can only be done on a Tile Palette. If Edit is not depressed, your actions can only be done on a Tilemap

<img src="{{ site.baseurl }}/images/BattleCity_UnityTilePalette1.png" alt="">

Adding of tile to the palette is very straightforward. Just drag and drop an image from your project window to the Tile Palette. 

Before you drag the image to the tile Palette, click on the image from the Project Window to access its <keyword>Import Settings</keyword>. Update the <keyword>Pixels Per Unit</keyword> to fit. If your image is a 256 X 256, set the image Pixels Per Unit to 256. This will ensure the painted tile will cover its entire tile and not be smaller or overlap. 

<div class="info">Try use a square image if possible to prevent unnecessary skewing</div>

You can also use a sprite sheet to create a tile but remember to factor in the divison of pixels in Pixels per unit due to the sprite sheet splitting into individual sprites.

Once you drag and drop, Unity will prompt you to save the tile asset. Once the tile is shown on the Tile Palette, use the paintbrush and depress the Edit button and paint the tile on the Tile Palette to form a 2 by 2 Square tile. This is to shorten the time for painting the tiles in the tilemap as i would be able to paint 4 tiles at one go as opposed to one. You do not have to do this but i find that makes things more convenient. The end result I have on my tile palette is shown as below.

<img src="{{ site.baseurl }}/images/BattleCity_UnityTilePalette2.png" alt="">
<div class="info">My dark grey tiles are 4 by 4 as they are for painting the Ground so I made it bigger to paint faster.</div>

### Painting the Boundary

Now we are ready to start painting! We need to gather information about how big our game area needs to be. From what I gathered from Wikipedia, Battle City Game Map is a 13 X 13 map size. But we need to take into account the width of a tank takes up the width of two bricks or two steel. Rather than trying to make our tile shrink to 0.5 unit, we should make the tile as 1 unit thus expanding the game area by double. So our game area should be of 26 X 26 units. This will make it easier to calculate position as it will increment by 1 instead of 0.5.

The first thing we need to do is create the boundary which will be used to constrain the gameplay in the 26 X 26 units area. 
1. Go to the tile palette and select the Active Tilemap as Boundary. 
2. Use the Select Tool from the Tile Palette(the icon that looks like the mouse arrow), use it to click on the grid in the Scene Window. The Inspector Window will change into the one for Grid Selection. Note the Position X and Y of the Grid Selection Inspector, continue to click and move your position on the Scene Window until it indicates -14 for Position X and -14 for Position Y. This is where the Boundary will be.

<img src="{{ site.baseurl }}/images/BattleCity_UnityTilePalette3.png" alt="">

<div class="info">Why -14, -14 is the boundary? We have reset the transform of the grid to origin. Its center tiles are at 0,0 and -1,0 (we are forming an even number size gameplay area so its center must be 2 tiles instead of 1)   Our We can then deduce the game area will be from -13, 12(top left corner) to 12, -13(bottom right corner) which makes the four corners of the boundary at -14,-14 and -14,13 and 13,13 and 13,-14.</div>

Once the correct position is selected, paint it with one of the tiles in the tile palette. Any tile is ok as it is just for visibility purposes. Do the same for the corners of -14,13 and 13,13 and 13,-14. Once you have painted the edges, paint to join the four corners forming a square which will be surrounding the game area.

<img src="{{ site.baseurl }}/images/BattleCity_UnityTilePalette4.png" alt="">

Once you have the boundary painted, select the Boundary Tilemap and add component <keyword>Tilemap -> Tilemap Collider 2D</keyword>. This will set the tiles painted in the Boundary with a collider each to prevent objects from passing through the boundary.

<img src="{{ site.baseurl }}/images/BattleCity_UnityScene2.png" alt="">

You would have noticed that the collider will form a highlighted structure around each of tile. The frames are the places where collider detection is present. In this case, there is no need for collider detection in the intersection between the painted tiles in Boundary Tilemap as nothing will be able to reach it without touching the entire frame collider detection, this represents wasted resources. 

Instead, we can create a collider detection only at the frames of the entire boundary. To do this, add another component to the Boundary Tilemap's Inspector Tile <keyword>Physics2d -> Composite Collider 2D</keyword>. Then at the Tilemap Collider 2D component, check the box for <keyword>Used by Composite</keyword> and hey presto, the collider is only formed around the frame of the entire boundary. 

You would have also noticed that upon adding the composite collider, Unity adds a Rigidbody 2D component to the Boundary Game Object as well. Set the Rigidbody 2D <keyword>Body Type</keyword> to <keyword>Static</keyword>, if not you will see your boundary falling off the screen or get pushed away when hit by any object. 

So that concludes the Boundary Tilemap section. 

<img src="{{ site.baseurl }}/images/BattleCity_UnityScene3.png" alt="">

### Painting the Ground

This will be the easiest of the lot. Just change the <keyword>Active Tilemap</keyword> to <keyword>Ground</keyword> and start painting the area surrounded by the Boundary with the dark grey tile(s). The result should be like the below. 

<div class="warning">Take note not to overlap the Boundary as its renderer will be disabled later which if the ground is overlapping will create an illusion that there is further ground to move.</div> 

<img src="{{ site.baseurl }}/images/BattleCity_UnityScene4.png" alt="">


### Painting the base

We will not be creating an entire stage this time. Instead we will just create the bare essential for the stage. Take a look at one of the stage, the only thing that is consistent in all the stages is the base (8 bricks surrounding the eagle).

<img src="{{ site.baseurl }}/images/BattleCity_BattleScene1.png" alt="">

So all we need to do to create the eagle and its surrounding bricks to complete the painting. 

<div class="warning">Remember to switch the Active Tilemap to Bricks before painting the bricks.</div> 

For the eagle, I do not suggest creating it on tilemap because it has a unique condition of Game Over when collided with a tank bullet so you can't use an existing tilemap to paint it and it is too much of overkill to create a tilemap just for this. 

So what I will do is just create a Game Object with a Sprite to render the eagle. You will need to change its <keyword>Pixel Per Unit</keyword> Settings correctly.

<div class="info">The Eagle is about the size of a tank, so you will need to set its Pixels Per Unit to half its actual size so that the sprite will cover 2 X 2 unit. Mine is a square image of 70 X 70 pixels so my Pixel Per Unit will be 35.</div> 

Once you have the appropriate Pixel Per Unit, drag and drop the Sprite from your Project Window to the Hierarchy. Unity will create the Game Object with a Sprite Renderer component automatically. Rename the Game Object to Eagle.

Add the <keyword>Box Collider 2D</keyword> ensuring that the collider forms a square entirely around the eagle. Create a script called Eagle and attach it to the Eagle as well. This script will handle the collision detection reactions and trigger of the Game Over Scenario.(we will discuss the code in later sections when we move on to the game completion scenarios)

Optional: You can create a broken flag Sprite and "child" it to the Eagle Game Object. Set it to disabled so that if the Eagle got destroyed by a tank bullet, you can just disable the Eagle Sprite Renderer and enable the child Broken flag Game Object (That is what is happening in Battle City).

Move the Eagle Game Object to the bottom Center of the Game Play Area. 

<div class="info">Tip: Just change its transform position to X: 0 and Y: -12</div> 

Then draw the Brick Tiles around the Eagle(remember to change the Active Tilemap to Bricks. Nagging again). You should have something like the below.

<img src="{{ site.baseurl }}/images/BattleCity_UnityScene5.png" alt="">


Now select the Brick Tilemap from the Hierarchy and add a Tilemap Collider 2D component in its Inspector to make the bricks a collision candidate. You DO NOT need to add composite collider 2D as there will be scenarios where there will be only a single brick tile instead of a congregation of bricks together.

### Adding the colliders

As all the tilemaps we are touching until now are the Ground, Boundary and Bricks. We have yet to use the Steel, Canopy, Water and TilePalette. I would want to add the necessary components(primarily <keyword>Tilemap Collider 2D</keyword>) to ensure it will work once we paint on those tilemaps. Canopy and TilePalette do not need any Collider so we can ignore it. But do go and add Tilemap Collider 2D to the Steel and Water Tilemaps.

<div class="info">Optional: You can create a script and add an Audio Source component on the Steel Tilemap for creating the sound effect of "striking the steel". If you do not need any sound, just ignore this.</div>

### Getting rid of Boundaries

Now you have added the necessary components to your Tilemaps. You do not need the Boundary anymore! Select the Boundary Tilemap from the Hierarchy and uncheck the Tilemap Renderer and the Boundary will magically disappear leaving the collider to stop any objects from leaving the Gameplay area.

<img src="{{ site.baseurl }}/images/BattleCity_UnityScene6.png" alt="">

### Become fab by making prefab

That should be all for the tilemap. The most important task now is not to save the scene, but to make this entire Scene a <keyword>prefab</keyword>. This will ensure it will become a template for you to use later to make more Stages. All you need to do is to drag and drop the Stage Essential Game Object (which also contains the tilemaps and Main Camera and Eagle) to the project window. Unity will create the prefab object for you. Move the prefab to your preferred folder for organization. Prefab your MasterTracker Game Object as well. Oh yeah, <keyword>save</keyword> your scene too.

#### Play the game! Not yet!

Now you can play the game. Sorry, just kidding. There is nothing to play. We are still missing the protagonist(the good guy tank) and antagonists(the villain tanks). We will talk about it in the [next post]({{ site.baseurl }}{% post_url 2018-03-07-battle-city-unity-4 %}).

{% include serieswithintro.html %}

