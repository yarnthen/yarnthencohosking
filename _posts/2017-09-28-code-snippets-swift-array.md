---
layout: post
title: Array related Code Snippets for Swift 4
excerpt: This post contains code snippets for Swift 4 with regards to array
summary: This post contains code snippets for Swift 4 with regards to array
categories: [Snippets]
tags: [Swift, iOS, Programming, Snippets]
comments: true
---

**Declare an empty array with its type stated**
{% highlight swift %}
var someInts = [Int]()							//empty array of integer
var someAnswers = [Bool]()						//empty array of boolean
var someString = [String]()						//empty array of string
{% endhighlight %}

**Declare an array containing the strings** 
{% highlight swift %}
var shoppingList = ["Eggs", "Milk", "Fish", "Chicken", "Apples]				//All items are string so Swift will auto cast it as a String Array
{% endhighlight %}
{% highlight swift %}
var shoppingList : [String] = ["Eggs", "Milk", "Fish", "Chicken", "Apples]  //Force declare it as a string array. If items added in this line is not a string, error will occur
{% endhighlight %}

**Some ways to manipulate array**
{% highlight swift %}
someInts.append(3) 										//add the integer 3 to the array at the last.
someInts += [3] 										//same as above another way to add the integer 3 to the array at the last
someInts += [3, 4, 5] 									//add multiple items to array at the last
someInts = [] 											//clear the array of items but it is still an array of its original type
var threeDoubles = Array(repeating: 0.0, count: 3) 		//creating an array of [0.0, 0.0, 0.0]. All items are Double so Swift will auto cast it as a Double Array
var sixDoubles = threeDoubles + threeDoubles 			//adding two arrays together, and equals [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
shoppingList[1...3] = ["Coconuts", "Banana"] 			// replace 3 items of position 2 to 4 in shoppingList array with 2 items Coconuts and Banana. So now array is from 5 items to 4 items
shoppingList.insert("Maple Syrup", at: 0) 				// add Maple Syrup in shoppingList array at position 0 (1st item)
shoppingList.remove(at: 0) 								// remove item at position 0 (1st item) of shoppingList array
shoppingList.removeLast() 								// remove last item of shoppingList array
shoppingList.sort() 									//sort shoppingList array in alphabetical order. If array contains numbers, it will be numerical order.
{% endhighlight %}

**Some ways to read array**
{% highlight swift %}
var Intt = someInts(0) 														//set Intt as the value of first item of someInts array
someInts.count 																//count number of items in array
shoppingList.isEmpty 														//check if array shoppingList is empty. If empty, will return boolean true
for product in shoppingList 												//for loop for items in shoppingList each loop will put the item's value to variable product
for (index, value) in shoppingList.enumerated() 							//for loop for items in shoppingList but wanting the value and position of item in array as well, value in product and position in index
{% endhighlight %}
