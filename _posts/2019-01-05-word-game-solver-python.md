---
layout: post
title: "Making a simple word game solver with python"
excerpt: Using Python to create a simple word game solver
summary: This post teaches how to create a word list generator for games like Word Cookie
categories: [How To]
tags: [Python, Words With Friends, Word Cookie, Word Streak]
comments: true
---

### Introduction
One of the most popular mobile games is Word Cookie which is a simple word game where you form words using different combinations of the random alphabets generated. It is a fantastic word skills improvement game that you can play during your free time.
<img src="{{ site.baseurl }}/images/wordcookie.jpg" alt="">
But I am someone who is not into these sort of games and distance myself away from these until my son approached me to help him in one of his session. Reluctantly I tried my hand on it but got stuck on finding words more than three alphabet characters! So I decided to create my script to help me in getting the words required. This does not improve my word skills, but at least it improves my programming skills. But don't tell my son I got a secret tool available to help me find words! 

I am on a Python programming learning journey, so I decided to use Python for this, but its concept should be able to be adapted to other programming language.

### Preparation
The first important task is to find all the words that are available. Thankfully there are people on the web who are kind enough to keep word lists such as [this from a GitHub repo](https://github.com/dwyl/english-words). You might want to do some sanity filter on the list as some words (AWOL, PVC) does not make sense. In the end, I gathered 341,111 words. Probably not foolproof as this is not the list that Word Cookie uses to generate its words but good enough to clear most game sessions and enhance my programming skill.

So now we are ready to start!

### High Level Explanation on program
Basically what the program will do is to:
1. Get all the words into a list from the file containing all the words
2. Get input of the alpha characters used to form the words
3. From the input in step 2, find the alphabets which are not used to form the words
4. Do a filter of the list in step 1 to remove any words which are longer than the number of alpha characters of the list created in step 2
5. Do another level of filter on top of the output from step 4 by removing any words containing alphabets not in the list created in step 2
6. Get all the possible permutation in arrangement of the characters of the list created in step 2
7. Compare the output of step 6 against the list of the two-level filtered english words from step 5
8. Print the output of step 7.


### Imports
To do the program, we will need to import two libraries. The string library for handling the string manipulation and the subset permutations of the itertools library to form all the permutation of the random characters.

{% highlight python%}
from itertools import permutations
import string
{% endhighlight %}

### Step 1: Get all the words from the file
The file I used is a text file with a word on every line. So open the file and read each line into a list called lines and do a rstrip to remove trailing and leading spaces and also the line feed carriage return. I also add a .lower command to set all the words to lowercase just in case. You can omit this part if you already got all words in lower case during preparation.
{% highlight python%}
lines = [line.rstrip('\n').lower() for line in open('words_reviewed.txt')]
{% endhighlight %}
### Step 2: Get input of the characters used to form words
Using the input command, you can get the characters into a list
{% highlight python%}
stringOfLetters = input("Key in the letters:")
{% endhighlight %}
### Step 3: Find the alphabets that are not used to form the words
What we need to do next is to find the alphabets which are not used. We will use the string.ascii_lowercase to get all the alphabets. We will then cast it as a set which we will use to subtract from the user inputted list of characters(which is also cast as a set) to get the missing alphabets.
{% highlight python%}
missingLetters = set(string.ascii_lowercase) - set(x for x in stringOfLetters)
{% endhighlight %}
### Step 4: Filter any words with length longer than the length of user inputted string
To make the script more efficient, we should reduce the number of words it need to compare. So the first part is to filter away any words that are longer than the number of characters as it is impossible for those words to be formed based on what is available.
{% highlight python%}
firstFilter = list(filter(lambda x: len(x) <= len(stringOfLetters), lines))
{% endhighlight %}
### Step 5: Second Filter to remove words containing unavailable characters
Same as step 4, it is impossible to create words with alphabets not available so we can also omit them as well. I used two for loops to first loop the step 4 filtered list and loop it against the missing characters. Those that are left over, we will write it to list secondFilter.
{% highlight python%}
secondFilter = []
found = False
for word in firstFilter:
    found = False
    for letter in missingLetters:
        if letter in word:
            found = True
            break
    if(found == False): 
        secondFilter.append(word)
{% endhighlight %}
### Step 6: Get all possible permutations in arrangement of the user inputted string
This is where the permutations function of itertools come in. We will get all possible permutation from 2 characters until the length of the inputted string and save it to a set so that there will no repeated entry.
{% highlight python%}
permutationSet=set()
for x in range(2,len(stringOfLetters)+1):
    possiblePermutations = permutations(stringOfLetters, x)
    for permutation in possiblePermutations:
        permutationSet.add("".join(permutation))
{% endhighlight %}
### Step 7: Compare all the possible permutation against the list of filtered english words
Now everything is straightforward. Just compare everything in the list and save the hits!
{% highlight python%}
Answers = []
for y in secondFilter:
    for element in permutationSet:
        if element == y:
            Answers.append(element)
{% endhighlight %}
### Step 8: Print the answer
{% highlight python%}
print(Answers)
{% endhighlight %}
### Summary
And there you have it, a simple word game solver created with python in less than 30 lines of code. This script can be better optimized, but it is definitely fit for purpose to play word cookie. Below is the full code, let me know if you have any comments or any improvements you can suggest!
{% highlight python%}
from itertools import permutations
import string
lines = [line.rstrip('\n').lower() for line in open('words_reviewed.txt')]
alphabet=set(string.ascii_lowercase)
stringOfLetters = input("Key in the letters:")
missingLetters = set(string.ascii_lowercase) - set(x for x in stringOfLetters) 
firstFilter = list(filter(lambda x: len(x) <= len(stringOfLetters), lines))
secondFilter = []
found = False
for word in firstFilter:
    found = False
    for letter in missingLetters:
        if letter in word:
            found = True
            break
    if(found == False): 
        secondFilter.append(word)
permutationSet=set()
for x in range(2,len(stringOfLetters)+1):
    possiblePermutations = permutations(stringOfLetters, x)
    for permutation in possiblePermutations:
        permutationSet.add("".join(permutation))
Answers = []
for y in secondFilter:
    for element in permutationSet:
        if element == y:
            Answers.append(element)
print(Answers)
{% endhighlight %}