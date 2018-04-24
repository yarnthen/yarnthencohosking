---
layout: post
title: "Recreate Battle City in Unity Part 25 : Bonus Crates - Shovel"
excerpt: Recreate Battle City in Unity Part 25
summary: This post is part 25 to the series of post detailing how I recreate Battle City in Unity
categories: [Tutorials]
tags: [Unity, Unity 3D, Battle City, NES, retro, tilemap, tile, tanks, gaming, classic]
comments: true
---

Hang in there; we are almost done with the tutorials. After that, you can enjoy creating your stages. Let's go and create the last bonus crate.

This should not be too difficult a task if we were to do it via the traditional way of using game objects to create the scene. But it becomes tricky because we build the scene using tilemaps; there appears to be no way to paint the tiles programmatically as we are unable to access the tile palette from code(at least I cannot find any). 

But Unity has allowed us to copy tiles from tilemap using code. So by doing creative usage of the <keyword>GetTile</keyword> and <keyword>SetTile</keyword> routines of <keyword>Tilemap</keyword> library, we can paint tiles with code. What we will do is to use a tilemap as a <keyword>Tile Palette</keyword>.(I created one at the beginning, now you know why I called it Tile Palette).

### Getting Tile Positions

The first thing we need to do is to determine the tile position of the bricks surrounding the eagle. That can be done by using the <keyword>Select</keyword> tool in the Tile Palette. For mine, it will be (-2,-13), (-2,-12), (-2,-11), (-1,-11), (0,-11), (1,-11), (1,-12), (1,-13).

### Creating the Tile Palette
Let's start by putting the TilePalette TileMap on top so that we can see it by changing its Order by Layer to 3. Change the Active Tilemap to TilePalette then paint the brick tile at position 0,0 and steel tile at location 1,1. Then change the Order to Layer to -2, so it will not appear on the scene.
<div class="info">The position you choose to paint doesn't matter. I selected mine for easy identification</div>
<img src="{{ site.baseurl }}/images/BattleCity_Spade_1.gif" alt="">

### Changing the tiles

We start by creating a script in the Grid Game Object calling it <keyword>GridMap</keyword>. Add the Library UnityEngine.Tilemaps at the top of the script.

{% highlight csharp%}
using UnityEngine.Tilemaps;
{% endhighlight %}

The concept is to remove the brick tiles at the location from Brick TileMap and replace it with Steel tiles on the Steel TileMap then vice-versus to revert. So we can create a single routine to do both by taking in the position, tile to change to and the tilemaps. Let's call this routine <keyword>UpdateAndRemoveTile</keyword>. The code as below.

{% highlight csharp%}
void UpdateAndRemoveTile(Vector3 position, TileBase tile, Tilemap tileMapToRemoveFrom, Tilemap tileMapToUpdate)
{   
    tileMapToRemoveFrom.SetTile(tileMapToRemoveFrom.WorldToCell(position), null);
    tileMapToUpdate.SetTile(tileMapToUpdate.WorldToCell(position), tile);
}
{% endhighlight %} 


### Getting the reference

To use the Tilemaps, we will need to get their reference in code. Declare 3 <keyword>TileMap</keyword> variables called <keyword>brickTileMap, steelTileMap and tilePaletteTileMap</keyword> with <keyword>SerializeField</keyword>. Then drag and drop the Bricks, Steel and TilePalette tilemap in the Inspector.
<img src="{{ site.baseurl }}/images/BattleCity_Spade_2.gif" alt="">

With this in, it will be easy to use the code below to trigger the UpdateAndRemoveTile routine.

{% highlight csharp%}
Vector3 steelTilePosition = new Vector3(1, 1, 0);
TileBase steelTile = tilePaletteTileMap.GetTile(tilePaletteTileMap.WorldToCell(steelTilePosition));
UpdateAndRemoveTile(new Vector3(-2,-13,0), steelTile, brickTileMap, steelTileMap)
{% endhighlight %}

<div class="info">Above is an example of updating the brick tile to steel tile. We first get the position of the steel tile in the TilePalette tilemap. Then use the <keyword>GetTile</keyword> routine to get the tile information. Then we pass the information to UpdateAndRemoveTile routine to do the hard work.</div>

We need to run the same command on the 6 tiles surrounding the eagle twice(once for turning brick to steel and then for steel to brick). We can split it into 3 routines to shorten the amount of code we need to write. 2 of the routines(<keyword>ChangeEagleWallToSteel and ChangeEagleWallToBrick</keyword>) will be to pass the tile info and tilemap into the routine(<keyword>UpdateTile</keyword>) which will run the UpdateAndRemoveTile 6 times on the position surrounding the eagle.

{% highlight csharp%}
void UpdateTile(TileBase tile, Tilemap tileMapToRemoveFrom, Tilemap tileMapToUpdate)
{
    UpdateAndRemoveTile(new Vector3(-2f, -13f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(-2f, -12f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(-2f, -11f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(-1f, -11f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(0f, -11f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(1f, -11f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(1f, -12f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    UpdateAndRemoveTile(new Vector3(1f, -13f, 0), tile, tileMapToRemoveFrom, tileMapToUpdate);
    tileMapToUpdate.gameObject.GetComponent<TilemapCollider2D>().enabled = false;
    tileMapToUpdate.gameObject.GetComponent<TilemapCollider2D>().enabled = true;
}
void ChangeEagleWallToSteel()
{
    Vector3 steelTilePosition = new Vector3(1f, 1f, 0);
    TileBase steelTile = tilePaletteTileMap.GetTile(tilePaletteTileMap.WorldToCell(steelTilePosition));
    UpdateTile(steelTile, brickTileMap, steelTileMap);
}
void ChangeEagleWallToBrick()
{
    Vector3 brickTilePosition = new Vector3(0f, 0f, 0);
    TileBase brickTile = tilePaletteTileMap.GetTile(tilePaletteTileMap.WorldToCell(brickTilePosition));
    UpdateTile(brickTile, steelTileMap, brickTileMap);
}
{% endhighlight %}

Finally we need to create a coroutine to have a timer rundown so that we can have the effect of the steel wall turn back to the brick wall after a period. Then have a Public routine to start the coroutine. This will allow the Spade Bonus Crate to trigger the routine to activate the coroutine.(remember a Game Object that is going to be destroyed cannot start a coroutine as it will cancel the coroutine as well)
{% highlight csharp%}
public void ActivateSpadePower()
{
    StartCoroutine(SpadePowerUpActivated());
}
IEnumerator SpadePowerUpActivated()
{
    ChangeEagleWallToSteel();
    yield return new WaitForSeconds(20f);
    ChangeEagleWallToBrick();
}
{% endhighlight %}

### Create the Spade Bonus Crate

Now we can create the Spade Bonus Crate. Start by dragging and dropping the Sprite you have for Spade bonus crate into the hierarchy which Unity will help to create the Game Object for you. Call the Game Object <keyword>Spade</keyword>.Add a <keyword>Box Collider 2D</keyword> and assure its boundary fills the entire crate's image, then make it a trigger. Also, assign <keyword>Powerups</keyword> layer to the Game Object. You should also set its <keyword>Order In Layer to 2</keyword> to ensure it gets rendered on top of all objects.

<div class="info">Adjust the sprite's Pixels Per Unit so that the Game Object fills the same amount of area as a tank.</div>

<img src="{{ site.baseurl }}/images/BattleCity_Spade_3.png" alt="">

Now we will create the code for the Spade crate. Add a new script called <keyword>Spade</keyword> to the Spade Game Object. Set the class to inherit from PowerUps. The full code for Spade as below.
{% highlight csharp%}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Spade : PowerUps
{
    GameObject grid;
    protected override void Start()
    {
        base.Start();
        grid = GameObject.Find("Grid");
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        GridMap gridmap = grid.GetComponent<GridMap>();
        gridmap.ActivateSpadePower();
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

<div class="info">The script finds the reference of the Grid Game Object to get to access the ActivateSpadePower routine in its script component GridMap.</div>

Now we are ready to try out!
<img src="{{ site.baseurl }}/images/BattleCity_Spade_4.gif" alt="">

Perfect! We are on a roll! Let's iron out a few details to make it more like Battle City. The first one will be the part where it is changing between steel wall and brick wall repetitively at the tail end of the timer before it completely changes back to the brick wall. To do this, we need to make ChangeEagleWallToSteel a coroutine and update SpadePowerUpActivated coroutine as below.

{% highlight csharp%}
IEnumerator SpadePowerUpActivated()
{
    StartCoroutine(ChangeEagleWallToSteel());
    yield return new WaitForSeconds(20f);
    ChangeEagleWallToBrick();
}
IEnumerator ChangeEagleWallToSteel()
{
    Vector3 steelTilePosition = new Vector3(1f, 1f, 0);
    Vector3 brickTilePosition = new Vector3(0f, 0f, 0);
    TileBase steelTile = tilePaletteTileMap.GetTile(tilePaletteTileMap.WorldToCell(steelTilePosition));
    TileBase brickTile = tilePaletteTileMap.GetTile(tilePaletteTileMap.WorldToCell(brickTilePosition));
    UpdateTile(steelTile, brickTileMap, steelTileMap);
    yield return new WaitForSeconds(15f);
    for (int i = 0; i < 5; i++){
        UpdateTile(brickTile, steelTileMap, steelTileMap);
        yield return new WaitForSeconds(0.5f);
        UpdateTile(steelTile, steelTileMap, steelTileMap);
        yield return new WaitForSeconds(0.5f);
    }
}
{% endhighlight %}

Let's try it again.

<img src="{{ site.baseurl }}/images/BattleCity_Spade_5.gif" alt="">

One item chalked off. Now to the last one. There are circumstances that a player might accidentally(cough...cough...foolishly) shoot at the steel wall with its Level 4 tank which will result in the steel wall to be damaged. In this case, the damaged part will not be filled back with bricks anymore. To create this effect, we will need to update an if statement to check if there is still a tile at a position in the UpdateAndRemoveTile routine.
{% highlight csharp%}
void UpdateAndRemoveTile(Vector3 position, TileBase tile, Tilemap tileMapToRemoveFrom, Tilemap tileMapToUpdate)
{
    if (!(tileMapToRemoveFrom == steelTileMap && tileMapToRemoveFrom.GetTile(tileMapToRemoveFrom.WorldToCell(position)) == null))
    {
        tileMapToRemoveFrom.SetTile(tileMapToRemoveFrom.WorldToCell(position), null);
        tileMapToUpdate.SetTile(tileMapToUpdate.WorldToCell(position), tile);
    }
}
{% endhighlight %}

Now let's do a full test on the effect.(I have shortened the Spade Bonus timer to 5 seconds for the testing) The spade should be able to:
1. Fill back any damaged brick tiles around the eagle
2. Have the flickering effect of brick and steel at the tail end of the Spade Bonus timer
3. Not have the ability to fill back the brick tiles at location where the steel tiles around the eagle are damaged.


<img src="{{ site.baseurl }}/images/BattleCity_Spade_6.gif" alt="">

That's all the Bonus Crates! Prefab the Spade crate and then update the GamePlayerManager with all the bonus crate prefabs.


That will be the end of my tutorial series for Battle City. There are of course other stuffs which I did not cover such as sounds and start menu. I want to get feedback on my tutorials from you all before working on those so that I can improve myself. So please put in the comments how I can improve! Thanks.

The game link for testing is here.