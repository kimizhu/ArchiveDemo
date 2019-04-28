# The Rarefied Heights of Mathematical Purity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/12/2004 5:43:00 PM

-----

A number of people have asked me for the software I used yesterday to extract the Google queries from the .TEXT referrer log.  Have patience, and all will be revealed. In this and the next blog entries I'm going to demonstrate the difference between computer **science** and computer **programming**.  In other words, we’re going to go from the **rarefied heights of pure mathematical proof** down to the **Stygian depths of quick-n-dirty script hacks**.   As an added bonus I'm once more going to pull the neat logical trick of [arguing both a statement and its opposite](http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx"). 

(SimpleScript is still coming, I promise.  I find a few minutes every day to work on the binder, but it is slow going.) 

Let me start today by giving you all a quick refresher on what we mean by the "**order**" of an algorithm.  Suppose you have a problem where the input can vary in size in some way.  For example, consider the problem "find the average of an array of numbers".  That's a pretty easy problem to solve: 

Total = 0  
For curNum = LBound(List) To UBound(List)  
  Total = Total + List(curNum)  
Next  
Average = Total / (UBound(List) - LBound(List) + 1) 

Clearly the amount of time it's going to take to find that average depends upon how long the list is\!  But what is the **relationship** between the size of the list and the amount of time it takes to find the average?  For this problem, it's pretty clear: you double the number of entries in the list, it's going to take about twice as long to run the algorithm.  Maybe not **exactly** twice as long, because of course, the first and last lines of this algorithm will run in the same amount of time no matter how big the list is.  But **asymptotically** -- as the list gets bigger and bigger -- the amount of time taken to run the algorithm depends almost entirely on the size of the input.  The little constant bits at the beginning and end become irrelevant compared to the cost of the loop. 

When I talk about the order of an algorithm, that's what I'm talking about -- how the algorithm behaves in the asymptotic case, without worrying about any of the little stuff.  I'm also not worried about the "constant factor" -- whether I could rewrite this loop to be twice as fast by being a more clever programmer or buying better hardware.  All I care about is a sort of *abstract* notion of **how the algorithm behaves when given really large inputs**. 

The "find the average" algorithm above is an Order-n algorithm, or O(n) for short.  That means that when you give it an input of size n, it takes some factor of n steps to run.  Whether each step takes a nanosecond or a millisecond depends on the hardware; for now I'm only concerned with the abstract fact that we've got a problem that gets **linearly harder** the more stuff you throw at it. 

Now consider the problem "find all duplicate entries in an array".  Here's a very naïve way to do it: 

For curNum = LBound(List) To UBound(List) - 1  
  For curNum2 = curNum + 1 To UBound(List)  
    If List(curNum) = List(curNum2) Then  
      Print "Duplicate " & List(curNum) & " at " & curNum & " " & curNum2  
    End If  
  Next  
Next

What's the order of this thing, given a list of size n?  Suppose n is 4.  The first time through the outer loop we're going to run the contents of the inner loop three times.  The second time through the outer loop, the inner loop contents run twice.  The third time, once.  In total, the comparison step runs 3 + 2 + 1 = 6 times.  The print step runs no more than that many times, so lets ignore it. Generalize that.  For a list of size n, the comparison runs (n - 1) + (n - 2) + … + 3 + 2 + 1 times, and if you remember Gauss's formula, you know that the total is (n<sup>2</sup> - n)/2.  The constant factor and linear term are irrelevant in the asymptotic case, so this is an O(n<sup>2</sup>) algorithm.  Make the input 10 times bigger and it takes 100 times longer to run, more or less. 

Let's get a little more complex.  What about sorting lists?  There are many different sorting algorithms, but MergeSort, BubbleSort, InsertionSort, QuickSort, all those sorts have a basic pattern in common.  They all use this algorithm schema:

1\)     Somehow pick two elements from the array.  
2\)     Compare them.  
3\)     Do something based on which of the two is the larger.  
4\)     Repeat until the array is sorted.

The way QuickSort picks the elements and acts upon them is very different from MergeSort of course, but the basic scheme is the same.  

Here's a computer science question:  **what is the best possible order for a sort algorithm**?  Can we make an O(n) sort algorithm out of this algorithm schema?  Turns out the answer is “no“.  Don't believe me?  I'll prove it to you. 

Let's consider how our generalized sort algorithm schema above sorts a list of three items, which, for the sake of simplicity, are "A", "B" and "C".  There are 3\! = 3 x 2 x 1 = 6 possible orderings (because there are three choices for the first item, two for the second, and that leaves only one choice for the third.) 

\<A, B, C\>  
\<A, C, B\>  
\<B, A, C\>  
\<B, C, A\>  
\<C, A, B\>  
\<C, B, A\> 

Suppose the generalized sort algorithm above is given the list \<a1, a2, a3\>.  How does it figure out which of the six cases above we're in?  Because obviously once you know which of those six cases you're in, the problem is solved.  You know what the permutation is, so you can unpermute it back to the ordered state.  

That might not be clear.  Let me state it a different way.  There are **six** possible **inputs** to the sort algorithm, but only **one** desired **output** -- the sorted list.  That means that the sort algorithm has to have at least six different **code paths**, where each path is based on the result of **comparing two elements**.  

Maybe that's still not clear.  Let's unroll the generalized sort algorithm above to make the **perfect sort algorithm for three elements**.  We'll have a** unique code path for every possible permutation**. 

if a1 \> a2 {  
  if a1 \> a3 {  
    if a2 \> a3 {  
      sorted order is \<a3, a2, a1\>  
    } else {  
      sorted order is \<a2, a3, a1\>  
    }  
  } else {  
    sorted order is \<a2, a1, a3\>  
  }  
} else {  
  if a2 \> a3 {  
    if a1 \> a3 {  
      sorted order is \<a3, a1, a2\>  
    } else {  
      sorted order is \<a1, a3, a2\>  
    }  
  } else {  
    sorted order is \<a1, a2, a3\>  
  }  
}

Of course this becomes a ludicrous algorithm for a large list, but surely you agree that **in theory** it doesn't get any more optimal than this algorithm, and that all the sorts I mentioned above are this algorithm in disguise, right? The only work that this algorithm does is comparisons.  How many do we have to make, worst case?  In this example the worst case requires three comparisons.  You can't possibly write a sort algorithm that makes fewer than **three comparisons, worst case**, to determine how to unscramble **three items**. 

Look at the structure of the algorithm.  It's a classic “divide and conquer”.  Each time we make a comparison, **we eliminate half** of the possible remaining permutations.  The first comparison, we go from six possibilities to three.  The next comparison we go from three possibilities to (worst case) two.  Next comparison, we're down to only one possible case, so we're done. Generalize it.  How many comparisons do we have to make with a longer list?  How deeply nested are those if statements going to get as the list gets larger?   

Three items in the list, worst case of three possible comparisons.  O(n), right?  Well, let’s see.  What if we add another item to the list, go from three to four.  Suddenly there are **four times** as many possible permutations, so we need **two more** comparisons.  And with five items in the list there are twenty times more permutations.  Hmm.  Maybe it isn't O(n).  What is the **minimum number of comparisons we need to make, in the worst case**?  Call that number j and lets reason backwards. Suppose we **only had that many comparisons to make**, how big a list could we handle? With at most j deep nested comparisons clearly we can build an optimal “nested if statement” sort algorithm that handles at most 2<sup>j</sup> cases.  For a list of size n, there are n\! permutations, and we need a unique code path per permutation. Therefore we need to know what the relationship is between j and n such that 2<sup>j</sup> \> n\!  That will tell us how many comparisons we need for a given list size. 

Now I'm going to pull a little handwaving trick here and pull a weak version of **Stirling's Approximation** out of thin air.  I can prove it later if you'd like, but for now, please accept the fact that for large values,   n\! \> (n/e)<sup>n</sup>  where e is the [estimated cash value of Google divided by one billion](http://news.com.com/2100-1024-5201978.html?part=dht&tag=ntop "http://news.com.com/2100-1024-5201978.html?part=dht&tag=ntop"). 

Therefore 

2<sup>j</sup> \> n\! \> (n/e)<sup>n</sup>   

Take the base two log of both sides, and we get 

j \> n log n - n log e 

And we're done.  The second term is asymptotically insignificant compared to the first, so we'll throw it away, and declare that any sort on a list of n items has at least a worst case of O(n log n) comparisons.   Our dream of a linear sort algorithm is dashed\! 

Using my ESP powers, I sense what you're thinking:  **"What the heck does this have to do with parsing your Google query log?"**  Tune in next time\!  In the next exciting episode of FABULOUS ADVENTURES, you'll hear Eric say: 

"As you can see, I've sorted the Google query keywords by frequency using a worst-case O(n) algorithm, thereby providing a counterexample to my proof from last time that such a sort algorithm is impossible." 

Will the very fabric of mathematics unravel?  Is Eric deluded?  Keep your feed aggregator tuned in and you'll find out.

