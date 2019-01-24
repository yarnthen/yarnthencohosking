---
layout: post
title: "Making a simple spiral matrix search with python"
excerpt: Using Python to create a simple spiral matrix search
summary: This post teaches how to create a simple spiral matrix search with Python
categories: [How To]
tags: [Python, Spiral, Outward , Search, Simple]
comments: true
---

### Introduction
I am in progress of creating a game where a team of game characters will seek out its nearest enemy in a grid setup. There are a lot of ways how this can be achieved. For example, one of the common ways is to keep track of all the enemies in a list and loop through the list to get the nearest enemy. But this is not feasible for me as this will be a multi-player game which means there is a lot of permutation of character vs enemy(each player team's enemy is everyone else group of game characters). So I have decided to
do it via checking surrounding grids and expanding outwards. 
### Preparation
To aid with this setup, I have a 2D list to store the position. As illustrated below(simplified) we can see that if the grid is occupied, it will be marked as a number other than zero. Zero means it is unoccupied. Different numbers mean they are enemy of one another. For example, a grid that contains 2 holds the enemy of the characters residing in the grid value containing 1 and vice versus.
<img src="{{ site.baseurl }}/images/spiral1.png" alt="">


## Explanation of the program

### 1st Layer of surrounding grids
Let's see how we are going to write the program starting with finding the pattern of the surrounding grids. We will begin with the first layer of surrounding grids and move out from there.
<img src="{{ site.baseurl }}/images/spiral2.png" alt="">

Let's take the grid we are going to find its surrounding grids to be X , Y. So the surrounding grids will be:
1. X - 1 , Y + 1
2. X - 1 , Y
3. X - 1 , Y - 1
4. X     , Y + 1
5. X     , Y - 1
6. X + 1 , Y + 1
7. X + 1 , Y
8. X + 1 , Y - 1

If you see the list of surrounding grids it is all of the combinations  +1 and -1. <br/>
In cases where are +0 (or nothing added/subtracted on X or Y), the other dimension will either be with a + 1 or - 1 (for example X, Y -1 or X + 1, Y). <br/>
And there is only addition and subtraction of one, not two or more. <br/>
So we can deduce that the addition and subtraction are in correspond to its layer. Let's call this layer number as <keyword>layerNumber</keyword>. So by substituting the + 1 and -1 with layerNumber we will get:

1. X - <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
2. X - <keyword>layerNumber</keyword> , Y
3. X - <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>
4. X , Y + <keyword>layerNumber</keyword>
5. X , Y - <keyword>layerNumber</keyword>
6. X + <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
7. X + <keyword>layerNumber</keyword> , Y
8. X + <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>

### 2nd layer of surrounding grids
Now is the exciting part when we get to the second layer.
<img src="{{ site.baseurl }}/images/spiral3.png" alt="">

We can now see the surrounding grids to be:
1. X - 2 , Y + 2
2. X - 2 , Y + 1
3. X - 2 , Y 
4. X - 2 , Y - 1
5. X - 2 , Y - 2
6. X - 1 , Y + 2
7. X - 1 , Y - 2
8. X     , Y + 2
9. X     , Y - 2
10. X + 1 , Y + 2
11. X + 1 , Y - 2
12. X + 2 , Y + 2
13. X + 2 , Y + 1
14. X + 2 , Y
15. X + 2 , Y - 1
16. X + 2 , Y - 2

```
```
Phew!! Much more than the 1st layer. Now we can see more combination of addition and subtraction. But the pattern we can see now even if there is a decrement of the addition or subtraction factor, the other dimension will always be an addition or subtraction of the layer number. So let's substitute the values of addition and subtractions which are lower than the layer number(if layer number is 2 so the values will be 1 and 0) as <span style="color:Red">decValue</span> and the values of addition and subtractions that are corresponding with the layer number as <keyword>layerNumber</keyword>:
1. X - <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
2. X - <keyword>layerNumber</keyword> , Y + <span style="color:Red">decValue</span>
3. X - <keyword>layerNumber</keyword> , Y +/- <span style="color:Red">decValue</span>
4. X - <keyword>layerNumber</keyword> , Y - <span style="color:Red">decValue</span>
5. X - <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>
6. X - <span style="color:Red">decValue</span> , Y + <keyword>layerNumber</keyword>
7. X - <span style="color:Red">decValue</span> , Y - <keyword>layerNumber</keyword>
8. X +/- <span style="color:Red">decValue</span>    , Y + <keyword>layerNumber</keyword>
9. X +/- <span style="color:Red">decValue</span>    , Y - <keyword>layerNumber</keyword>
10. X + <span style="color:Red">decValue</span> , Y + <keyword>layerNumber</keyword>
11. X + <span style="color:Red">decValue</span> , Y - <keyword>layerNumber</keyword>
12. X + <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
13. X + <keyword>layerNumber</keyword> , Y + <span style="color:Red">decValue</span>
14. X + <keyword>layerNumber</keyword> , Y +/- <span style="color:Red">decValue</span>
15. X + <keyword>layerNumber</keyword> , Y - <span style="color:Red">decValue</span>
16. X + <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>

Note that those which do not have addition or subtraction values are replaced with +/- decValue because they are still part of decValue(0 is smaller than the layer Number), it is just that we do not know if it is addition and subtraction so we will put both addition and subtraction symbol to it. Now let's rearrange the list so that those with only layer Numbers will group together.

1. X - <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
2. X - <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>
3. X + <keyword>layerNumber</keyword> , Y + <keyword>layerNumber</keyword>
4. X + <keyword>layerNumber</keyword> , Y - <keyword>layerNumber</keyword>
5. X - <keyword>layerNumber</keyword> , Y + <span style="color:Red">decValue</span>
6. X - <keyword>layerNumber</keyword> , Y - <span style="color:Red">decValue</span>
7. X + <keyword>layerNumber</keyword> , Y + <span style="color:Red">decValue</span>
8. X + <keyword>layerNumber</keyword> , Y - <span style="color:Red">decValue</span>
9. X - <span style="color:Red">decValue</span> , Y + <keyword>layerNumber</keyword>
10. X - <span style="color:Red">decValue</span> , Y - <keyword>layerNumber</keyword>
11. X + <span style="color:Red">decValue</span> , Y + <keyword>layerNumber</keyword>
12. X + <span style="color:Red">decValue</span> , Y - <keyword>layerNumber</keyword>
13. X +/- <span style="color:Red">decValue</span>    , Y + <keyword>layerNumber</keyword>
14. X +/- <span style="color:Red">decValue</span>    , Y - <keyword>layerNumber</keyword>
15. X - <keyword>layerNumber</keyword> , Y +/- <span style="color:Red">decValue</span>
16. X + <keyword>layerNumber</keyword> , Y +/- <span style="color:Red">decValue</span>

### Forming the program (Part 1)
The first part is for those which only have layer number for addition and subtraction(1-4), this code will be as below:
{% highlight python%}
for layerNumber in range(1,maxNumberOfLayers):
    if (contains_enemy(X - layerNumber, Y + layerNumber)) list.add(X - layerNumber, Y + layerNumber)
    if (contains_enemy(X - layerNumber, Y - layerNumber)) list.add(X - layerNumber, Y - layerNumber)
    if (contains_enemy(X + layerNumber, Y + layerNumber)) list.add(X + layerNumber, Y + layerNumber)
    if (contains_enemy(X + layerNumber, Y - layerNumber)) list.add(X + layerNumber, Y - layerNumber)
    if (len(list)>0) break
{% endhighlight %}
The above is to loop through all the possible layers starting from the 1st and check the corresponding neighbor grid if it contains enemy(simplified as a function named contains_enemy), if it is true it will add to a list. Then exit the for loop if anything is added to the list as we only need to get the nearest enemies list which is the nearest possible layer away from the character.
### Forming the program (Part 2)
Next is handle those with the decValues(5-12). Since decValues are values lower than that of the layerNumber, we can use a for loop based on what is in the layerNumber.
{% highlight python%}
for layerNumber in range(1,maxNumberOfLayers):
    for decValue in range(layerNumber):
        if (contains_enemy(X - layerNumber, Y + decValue)) list.add(X - layerNumber, Y + decValue)
        if (contains_enemy(X - layerNumber, Y - decValue)) list.add(X - layerNumber, Y - decValue)
        if (contains_enemy(X + layerNumber, Y + decValue)) list.add(X + layerNumber, Y + decValue)
        if (contains_enemy(X + layerNumber, Y - decValue)) list.add(X + layerNumber, Y - decValue)
        if (contains_enemy(X - decValue, Y + layerNumber)) list.add(X - decValue, Y + layerNumber)
        if (contains_enemy(X - decValue, Y - layerNumber)) list.add(X - decValue, Y - layerNumber)
        if (contains_enemy(X + decValue, Y + layerNumber)) list.add(X + decValue, Y + layerNumber)
        if (contains_enemy(X + decValue, Y - layerNumber)) list.add(X + decValue, Y - layerNumber)
    if (len(list)>0) break
{% endhighlight %}
### Forming the program (Part 3)
Of course, I have yet to add in the part where decValue is equal to zero(13-16) to the code. There is no need to because it is already in the code(0 for decValue is stated as +/- decValue, I have already have + decValue and -decValue in the code). But you will notice there will be an issue if decValue is 0. There will be duplicates because if decValue is 0, X + decValue will be equal to X - decValue, same for Y. So we will need to sacrifice a set of the equations(e.g choose one from X + decValue and X - decValue) to be omitted if decValue is equal to zero to prevent duplication. Being a positive guy, I have decided to sacrifice the negative equations(those with subtraction). So this will be the end result:
{% highlight python%}
for layerNumber in range(1,maxNumberOfLayers):
    for decValue in range(layerNumber):
        if (contains_enemy(X - layerNumber, Y + decValue)) list.add(X - layerNumber, Y + decValue)
        if (contains_enemy(X + decValue, Y + layerNumber)) list.add(X + decValue, Y + layerNumber)
        if (contains_enemy(X + decValue, Y - layerNumber)) list.add(X + decValue, Y - layerNumber)
        if (contains_enemy(X + layerNumber, Y + decValue)) list.add(X + layerNumber, Y + decValue)
        if (decValue != 0):
            if (contains_enemy(X - layerNumber, Y - decValue)) list.add(X - layerNumber, Y - decValue)
            if (contains_enemy(X + layerNumber, Y - decValue)) list.add(X + layerNumber, Y - decValue)
            if (contains_enemy(X - decValue, Y + layerNumber)) list.add(X - decValue, Y + layerNumber)
            if (contains_enemy(X - decValue, Y - layerNumber)) list.add(X - decValue, Y - layerNumber)
    if (len(list)>0) break
{% endhighlight %}
### Forming the program (Finale)
Now to merge everything, we have:
{% highlight python%}
for layerNumber in range(1,maxNumberOfLayers):
    if (contains_enemy(X - layerNumber, Y + layerNumber)) list.add(X - layerNumber, Y + layerNumber)
    if (contains_enemy(X - layerNumber, Y - layerNumber)) list.add(X - layerNumber, Y - layerNumber)
    if (contains_enemy(X + layerNumber, Y + layerNumber)) list.add(X + layerNumber, Y + layerNumber)
    if (contains_enemy(X + layerNumber, Y - layerNumber)) list.add(X + layerNumber, Y - layerNumber)
    for decValue in range(layerNumber):
        if (contains_enemy(X - layerNumber, Y + decValue)) list.add(X - layerNumber, Y + decValue)
        if (contains_enemy(X + decValue, Y + layerNumber)) list.add(X + decValue, Y + layerNumber)
        if (contains_enemy(X + decValue, Y - layerNumber)) list.add(X + decValue, Y - layerNumber)
        if (contains_enemy(X + layerNumber, Y + decValue)) list.add(X + layerNumber, Y + decValue)
        if (decValue != 0):
            if (contains_enemy(X - layerNumber, Y - decValue)) list.add(X - layerNumber, Y - decValue)
            if (contains_enemy(X + layerNumber, Y - decValue)) list.add(X + layerNumber, Y - decValue)
            if (contains_enemy(X - decValue, Y + layerNumber)) list.add(X - decValue, Y + layerNumber)
            if (contains_enemy(X - decValue, Y - layerNumber)) list.add(X - decValue, Y - layerNumber)
    if (len(list)>0) break
{% endhighlight %}
### Summary
There you have it, the code, not very pythonic, but I believe this better explains how the spiral matrix search is achieved.