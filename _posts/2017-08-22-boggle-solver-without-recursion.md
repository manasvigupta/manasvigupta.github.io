---
layout: post
title: "Boggle solver without Recursion"
description: "In which I show code that solves Boogle (game) without using any recursion"
modified: 2017-08-22 00:02:36 +0530
#category: [blog]
tags: [data-structures, algorithms]
#image:
#  feature: texture-feature-06.jpg
#  credit: coffee #17 by Mariantonietta Continenza
#  creditlink: http://www.flickr.com/photos/42897741@N05/3987243989/
comments: true
share: 
---

### What is Boggle?

As per [wikipedia], Boggle is a word game played using a plastic grid of lettered dice, in which players attempt to find words in sequences of adjacent letters.  

<br/>
Essentially, when the game starts you get a matrix of letters like so:

{% highlight text %}
F X I E
A M L O
E W B X
A S T U
{% endhighlight %}

The goal of the game is to find as many words as you can that can be formed by chaining letters together. You can start with any letter, and all the letters that surround it are fair game, and then once you move on to the next letter, all the letters that surround that letter are fair game, except for any previously used letters. So in the grid above, for example, I could come up with the words LOB, TUX, SEA, FAME, etc. 

### Typical solution & challenge

Typical solutions use a recursion to solve this game. Obviously, recursion will be limited by stack depth.

### Solve without recursion

Inspired by this [Stackoverflow] thread, I solved this puzzle using Trie & good old Breadth first search algorithm.

<br/>
You can find code at following [github] page.

<br/>
The fastest solution you're going to get will probably involve storing your dictionary in a trie. Then, create a queue of triplets (x, y, s), where each element in the queue corresponds to a prefix s of a word which can be spelled in the grid, ending at location (x, y). Initialize the queue with N x N elements (where N is the size of your grid), one element for each square in the grid. Then, the algorithm proceeds as follows:

{% highlight text %}
While the queue is not empty:
  Dequeue a triple (x, y, s)
  For each square (x', y') with letter c adjacent to (x, y):
    If s+c is a word, output s+c
    If s+c is a prefix of a word, insert (x', y', s+c) into the queue
{% endhighlight %}

If you store your dictionary in a trie, testing if s+c is a word or a prefix of a word can be done in constant time (provided you also keep some extra metadata in each queue datum, such as a pointer to the current node in the trie), so the running time of this algorithm is O(number of words that can be spelled).

[wikipedia]:https://en.wikipedia.org/wiki/Boggle
[Stackoverflow]:https://stackoverflow.com/questions/746082/how-to-find-list-of-possible-words-from-a-letter-matrix-boggle-solver
[github]:https://github.com/manasvigupta/data-structure-algo/tree/master/src/main/java/bogglesolver
