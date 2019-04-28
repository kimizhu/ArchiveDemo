<div id="page">

# The Rarefied Heights of Mathematical Purity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/12/2004 5:43:00 PM

-----

<div id="content">

<div>

<span>A number of people have asked me for the software I used yesterday to extract the Google queries from the .TEXT referrer log.  Have patience, and all will be revealed.</span> <span>In this and the next blog entries I'm going to demonstrate the difference between computer **<span>science</span>** and computer **<span>programming</span>**.  In other words, we’re going to go from the **rarefied heights of pure mathematical proof** down to the **Stygian depths of quick-n-dirty script hacks**.  </span> <span>As an added bonus I'm once more going to pull the neat logical trick of [arguing both a statement and its opposite](http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx"). </span>

<span></span>

<span>(SimpleScript is still coming, I promise.  I find a few minutes every day to work on the binder, but it is slow going.) </span>

<span></span>

<span>Let me start today by giving you all a quick refresher on what we mean by the "**<span>order</span>**" of an algorithm.  Suppose you have a problem where the input can vary in size in some way.  For example, consider the problem "find the average of an array of numbers".  That's a pretty easy problem to solve: </span>

<span></span>

<span>Total = 0  
</span><span>For curNum = LBound(List) To UBound(List)  
</span><span>  Total = Total + List(curNum)  
</span><span>Next  
</span><span>Average = Total / (UBound(List) - LBound(List) + 1) </span>

<span></span>

<span>Clearly the amount of time it's going to take to find that average depends upon how long the list is\!  But what is the **<span>relationship</span>** between the size of the list and the amount of time it takes to find the average?  For this problem, it's pretty clear: you double the number of entries in the list, it's going to take about twice as long to run the algorithm.  Maybe not **<span>exactly</span>** twice as long, because of course, the first and last lines of this algorithm will run in the same amount of time no matter how big the list is.  But **<span>asymptotically</span>** -- as the list gets bigger and bigger -- the amount of time taken to run the algorithm depends almost entirely on the size of the input.  The little constant bits at the beginning and end become irrelevant compared to the cost of the loop. </span>

<span></span>

<span>When I talk about the order of an algorithm, that's what I'm talking about -- how the algorithm behaves in the asymptotic case, without worrying about any of the little stuff.  I'm also not worried about the "constant factor" -- whether I could rewrite this loop to be twice as fast by being a more clever programmer or buying better hardware.  All I care about is a sort of *abstract* notion of **<span>how the algorithm behaves when given really large inputs</span>**. </span>

<span></span>

<span>The "find the average" algorithm above is an </span><span>Order-n</span><span> algorithm, or </span><span>O(n)</span><span> for short.  That means that when you give it an input of size </span><span>n</span><span>, it takes some factor of </span><span>n</span><span> steps to run.  Whether each step takes a nanosecond or a millisecond depends on the hardware; for now I'm only concerned with the abstract fact that we've got a problem that gets **linearly harder** the more stuff you throw at it. </span>

<span></span>

<span>Now consider the problem "find all duplicate entries in an array".  Here's a very naïve way to do it: </span>

<span></span>

<span>For curNum = LBound(List) To UBound(List) - 1  
</span><span>  For curNum2 = curNum + 1 To UBound(List)  
</span><span>    If List(curNum) = List(curNum2) Then  
      </span><span>Print "Duplicate " & List(curNum) & " at " & curNum & " " & curNum2  
    End If  
  </span><span>Next  
</span><span>Next</span>

<span></span>

<span>What's the order of this thing, given a list of size </span><span>n</span><span>?  Suppose </span><span>n</span><span> is 4.  The first time through the outer loop we're going to run the contents of the inner loop three times.  The second time through the outer loop, the inner loop contents run twice.  The third time, once.  In total, the comparison step runs </span><span>3 + 2 + 1 = 6</span><span> times.  The print step runs no more than that many times, so lets ignore it.</span> <span>Generalize that.  For a list of size </span><span>n</span><span>, the comparison runs </span><span>(n - 1) + (n - 2) + … + 3 + 2 + 1</span><span> times, and if you remember Gauss's formula, you know that the total is </span><span>(n<sup>2</sup> - n)/2</span><span>.  The constant factor and linear term are irrelevant in the asymptotic case, so this is an </span><span>O(n<sup>2</sup>) </span><span>algorithm.  Make the input 10 times bigger and it takes 100 times longer to run, more or less. </span>

<span></span>

<span>Let's get a little more complex.  What about sorting lists?  There are many different sorting algorithms, but MergeSort, BubbleSort, InsertionSort, QuickSort, all those sorts have a basic pattern in common.  They all use this algorithm schema:</span>

<span></span><span><span>1)<span>     </span></span></span><span>Somehow pick two elements from the array.  
</span><span><span>2)<span>     </span></span></span><span>Compare them.  
</span><span><span>3)<span>     </span></span></span><span>Do something based on which of the two is the larger.  
</span><span><span>4)<span>     </span></span></span><span>Repeat until the array is sorted.</span>

<span></span>

<span>The way QuickSort picks the elements and acts upon them is very different from MergeSort of course, but the basic scheme is the same.  </span>

<span></span>

<span>Here's a computer science question:  **what is the best possible order for a sort algorithm**?  Can we make an </span><span>O(n) </span><span>sort algorithm out of this algorithm schema?  Turns out the answer is “no“.  Don't believe me?  I'll prove it to you. </span>

<span></span>

<span>Let's consider how our generalized sort algorithm schema above sorts a list of three items, which, for the sake of simplicity, are "A", "B" and "C".  There are </span><span>3\! = 3 x 2 x 1 = 6</span><span> possible orderings (because there are three choices for the first item, two for the second, and that leaves only one choice for the third.) </span>

<span></span>

<span>\<A, B, C\>  
</span><span>\<A, C, B\>  
</span><span>\<B, A, C\>  
</span><span>\<B, C, A\>  
</span><span>\<C, A, B\>  
</span><span>\<C, B, A\> </span>

<span></span>

<span>Suppose the generalized sort algorithm above is given the list </span><span>\<a1, a2, a3\>.</span><span>  How does it figure out which of the six cases above we're in?  Because obviously once you know which of those six cases you're in, the problem is solved.  You know what the permutation is, so you can unpermute it back to the ordered state.  </span>

<span></span>

<span>That might not be clear.  Let me state it a different way.  There are **six** possible **inputs** to the sort algorithm, but only **one** desired **output** -- the sorted list.  That means that the sort algorithm has to have at least six different **code paths**, where each path is based on the result of **comparing two elements**.  </span>

<span></span>

<span>Maybe that's still not clear.  Let's unroll the generalized sort algorithm above to make the **<span>perfect sort algorithm for three elements</span>**.  We'll have a**<span> unique code path for every possible permutation</span>**. </span>

<span></span>

<span>if a1 \> a2 {  
</span><span>  if a1 \> a3 {  
</span><span>    if a2 \> a3 {  
      </span><span>sorted order is \<a3, a2, a1\>  
    } </span><span>else</span><span> {  
      </span><span>sorted order is \<a2, a3, a1\>  
    }  
</span><span>  } else {  
</span><span>    sorted order is \<a2, a1, a3\>  
  }</span><span>  
</span><span>} else {  
</span><span>  if a2 \> a3 {  
</span><span>    if a1 \> a3 {  
      </span><span>sorted order is \<a3, a1, a2\>  
    }</span><span> else {  
      </span><span>sorted order is \<a1, a3, a2\>  
    }  
</span><span>  } else {  
</span><span>    sorted order is \<a1, a2, a3\>  
  }  
}</span><span></span>

<span>Of course this becomes a ludicrous algorithm for a large list, but surely you agree that **<span>in theory</span>** it doesn't get any more optimal than this algorithm, and that all the sorts I mentioned above are this algorithm in disguise, right?</span> <span>The only work that this algorithm does is comparisons.  How many do we have to make, worst case?  In this example the worst case requires three comparisons.  You can't possibly write a sort algorithm that makes fewer than **three comparisons, worst case**, to determine how to unscramble **three items**. </span>

<span></span>

<span>Look at the structure of the algorithm.  It's a classic “divide and conquer”.  Each time we make a comparison, **we eliminate half** of the possible remaining permutations.  </span><span>The first comparison, we go from six possibilities to three.  The next comparison we go from three possibilities to (worst case) two.  Next comparison, we're down to only one possible case, so we're done.</span> <span>Generalize it.  How many comparisons do we have to make with a longer list?  How deeply nested are those <span>if</span><span> statements going to get as the list gets larger?  </span> </span>

<span></span>

<span>Three items in the list, worst case of three possible comparisons.  </span><span>O(n)</span><span>, right?  Well, let’s see.  What if we add another item to the list, go from three to four.  Suddenly there are **<span>four times</span>** as many possible permutations, so we need **<span>two more</span>** comparisons.  And with five items in the list there are twenty times more permutations.  Hmm.  Maybe it isn't </span><span>O(n)</span><span>.  What is the **<span>minimum number of comparisons we need to make, in the worst case</span>**?  Call that number </span><span>j </span><span>and lets reason backwards. Suppose we **only had that many comparisons to make**, how big a list could we handle?</span> <span>With at most </span><span>j</span><span> deep nested comparisons clearly we can build an optimal “nested if statement” sort algorithm that handles at most </span><span>2<sup>j</sup></span><span> cases.  For a list of size </span><span>n</span><span>, there are </span><span>n\!</span><span> permutations, and we need a unique code path per permutation. Therefore we need to know what the relationship is between </span><span>j</span><span> and </span><span>n</span><span> such that </span><span>2<sup>j</sup> \> n\!</span><span>  That will tell us how many comparisons we need for a given list size. </span>

<span></span>

<span>Now I'm going to pull a little handwaving trick here and pull a weak version of **Stirling's Approximation** out of thin air.  I can prove it later if you'd like, but for now, please accept the fact that for large values,   </span><span>n\! \> (n/e)<sup>n</sup></span><span>  where </span><span>e</span><span> is the [estimated cash value of Google divided by one billion](http://news.com.com/2100-1024-5201978.html?part=dht&tag=ntop "http://news.com.com/2100-1024-5201978.html?part=dht&tag=ntop"). </span>

<span></span>

<span>Therefore </span>

<span></span>

<span>2<sup>j</sup> \> n\! \> (n/e)<sup>n</sup></span><span>  </span><span> </span>

<span></span>

<span>Take the base two log of both sides, and we get </span>

<span></span>

<span></span>

<span>j \> n log n - n log e </span>

<span></span>

<span>And we're done.  The second term is asymptotically insignificant compared to the first, so we'll throw it away, and declare that any sort on a list of </span><span>n</span><span> items has at least a worst case of </span><span>O(n log n)</span><span> comparisons.  </span> <span>Our dream of a linear sort algorithm is dashed\! </span>

<span></span>

<span>Using my ESP powers, I sense what you're thinking:  **<span>"What the heck does this have to do with parsing your Google query log?"</span>**  Tune in next time\!  In the next exciting episode of FABULOUS ADVENTURES, you'll hear Eric say: </span>

<span></span>

<span>"As you can see, I've sorted the Google query keywords by frequency using a worst-case O(n) algorithm, thereby providing a counterexample to my proof from last time that such a sort algorithm is impossible." </span>

<span></span>

<span>Will the very fabric of mathematics unravel?  Is Eric deluded?  Keep your feed aggregator tuned in and you'll find out. </span>

<span></span>

<span></span>

<span></span>

<span></span>

<span></span>

<span></span>

<span></span>

<span></span>

<span></span>

</div>

</div>

</div>

