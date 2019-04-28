# Recursion and Dynamic Programming

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/21/2004 9:43:00 AM

-----

 Back in May [we were discussing the merits and drawbacks of recursive programming techniques](http://blogs.msdn.com/ericlippert/archive/2004/05/20/136327.aspx "http://blogs.msdn.com/ericlippert/archive/2004/05/20/136327.aspx") -- that is, writing functions which break down problems into sub-problems, and then call themselves.  

 The drawback of such an approach is that in some cases, you end up doing way more work than necessary.  The example I used was the naïve Fibonacci algorithm: 

 function fib(n) {  
      if (n == 1 || n == 2)  
            return 1;  
      return fib(n-1) + fib(n-2);  
} 

 This is an **exponential** algorithm -- calculating the first few numbers is very cheap, but once we get up into the tens and twenties, you very quickly end up doing millions of recursive steps.  Something that you might notice is that since fib(20) cannot possibly need to calculate more than 19 distinct previous fib values, that most of those millions of calculations are the same ones being done over and over again.   (ASIDE: This seemingly obvious argument has a name: the **[Pigeonhole Principle](http://en.wikipedia.org/wiki/Pigeonhole_principle "http://en.wikipedia.org/wiki/Pigeonhole_principle")**.) 

 Therefore, there's a way to keep this recursive AND make it efficient: don't recompute a result you already know\!  How about this? 

 var fibarr = new Array(0, 1, 1);  
function fib(n) {  
      if (fibarr\[n\] == null)  
            fibarr\[n\] = fib(n-1) + fib(n-2);  
      return fibarr\[n\];  
} 

 All of a sudden our formerly exponential algorithm has become linear if n has not already been calculated, and constant if it has. The time performance gets a huge win, but does so at a cost -- **it chews up memory to store the results**.  It's a pretty small cost though compared to the win. 

 This technique of taking an inefficient algorithm and improving its performance by storing and reusing earlier results is called **[memoization](http://www.nist.gov/dads/HTML/memoize.html "http://www.nist.gov/dads/HTML/memoize.html")** or **[dynamic programming](http://www.nist.gov/dads/HTML/dynamicprog.html "http://www.nist.gov/dads/HTML/dynamicprog.html")**.  Dynamic programming isn't hard, but a lot of script programmers don't know about this powerful technique, so I thought I might talk about it a bit today. 

 (ASIDE: Some would draw the subtle distinction that "dynamic programming" specifically means “using memoization to solve *optimization* problems” -- but I wouldn't.  Note also that this term predates its use in the context of *computer* programming.  The mathematician Richard Bellman originated the term in the 1950's -- he was studying the behaviour of [Markovian systems](http://www-anw.cs.umass.edu/~rich/book/4/node1.html "http://www-anw.cs.umass.edu/~rich/book/4/node1.html").  However, for my purposes I'm going to ignore the historical link between dynamic programming and the probability problems it was invented to solve, and talk just about the recursive programming applications of the technique.) 

 I started thinking about this because recently someone asked me how "diff" algorithms work -- you know, those programs that take two text files and show you what the differences and commonalities are.  There are lots of different "diff" algorithms, and some of them use dynamic programming techniques. 

 Here's a related problem: determine the **longest common subsequence** of two arrays.  Let me clarify what exactly I mean by that.  Consider the following lists: 

 var original = new Array("jackdaws", "love", "my", "big", "sphinx", "of", "quartz");  
var modified = new Array("some", "big", "jackdaws", "love", "my", "sphinx",  "of", "quartz"); 

 We want the **longest** sequence of items that are found in both sets **in the same order**. Here the longest common subsequence would be "jackdaws", "love", "my", "sphinx", "of", "quartz".   The "big" isn't in the same order with respect to the rest of the subsequence, so it does not appear in the longest common subsequence. 

 I'm sure you can see how this pertains to the "diff" problem -- if you know the longest common subsequence then it makes sense that it would be the "in common" part, and everything that is not in the longest common subsequence is in the "things that have changed" part.  (Of course, diff algorithms usually operate on lines of text, not on individual words, but the difference is irrelevant, so we'll stick with words for the sake of the example.  Also, there are more sophisticated diff algorithms, which I won't go into today.) 

 How do we solve this problem?  We could start by noticing that **if the two sequences** **start with the same word**, **then the longest common subsequence always contains that word**.  We can automatically put that word on our list, and we've just reduced the problem to finding the longest common subset of the rest of the two lists.  We've made the problem smaller, which is goodness.  

 But if the two lists do not begin with the same word, then one or both of them is not on the longest list.  But one of them might be. How do we determine which one, if any, to add?  Try it both ways and see\!  Either way, the two sub-problems are manipulating smaller lists, so we know that the recursion will eventually terminate.  Whichever trial results in the longer common subsequence is the winner. 

 Instead of "throwing it away" by deleting the item from the array, instead let's just write a routine that finds the longest common subsequence that exists **after a particular starting point in both arrays**. 

 Here's a solution without memoization: 

 function LongestCommonSubsequence(arr1, arr2)  
{  
      return LCS(0, 0);  
  
function LCS(start1, start2)  
{  
      var result;  
      var remainder1;  
      var remainder2;  
  
// If we're at the end of either list, then the longest subsequence is empty 

       if (start1 == arr1.length || start2 == arr2.length)  
            result = new Array();  
      else if (arr1\[start1\] == arr2\[start2\])  
      {  
  
// If the start element is the same in both, then it is on the LCS, so  
// we'll just recurse on the remainder of both lists.  
  
            result = new Array();  
            result\[0\] = arr1\[start1\];  
            result = result.concat(LCS(start1 + 1, start2 + 1));  
      }  
      else  
      {  
  
// We don't know which list we should discard from.   
// Try both ways, pick whichever is better.  
  
            remainder1 = LCS(start1 + 1, start2);  
            remainder2 = LCS(start1, start2 + 1);  
            if (remainder1.length \> remainder2.length)  
                  result = remainder1;  
            else  
                  result = remainder2;  
      }  
  
      return result;  
}  
} 

 

 This algorithm works, but holy cow is it ever inefficient\!  If the two arrays are both around n items long then this turns out to be exponential, for the same reason that the naïve fib implementation was exponential**.  It recalculates the same subsequences over and over again**.  In this particular example there are 2410 recursions to find the LCS of a seven and an eight item array\!  Clearly there are only 56 possible "start1" and "start2" combinations to try, so by the Pigeonhole Principle there must be *massive* re-calculation going on here.  If the arrays get longer, this algorithm blows up enormously. 

 We can do a lot better, using the same technique as we used above.  Every time we solve a subproblem, we make a note of the solution. 

 function LongestCommonSubsequence(arr1, arr2)  
{  
      var solutions = {};  
      return LCS(0, 0);  
  
function LCS(start1, start2)  
{  
      var result;  
      var remainder1;  
      var remainder2;  
      var index = start1 + "," + start2;  
      if (solutions\[index\] \!= null)  
            return solutions\[index\];  
  
      // \[blah blah blah, same as above\]  
  
      solutions\[index\] = result;  
      return result;  
}  
}  Which only does 82 recursions. 

 If you were *really* being clever, you might notice that the recursive solution **eventually fills in almost the entire solution array**, and only ever looks "up" from the current position, so you might as well simply write an **iterative** program that fills in the solution array **backwards**.   A downside of this particular implementation is that it chews up rather a lot of memory -- on the order of n^2 extra arrays in the solutions table, many of which are potentially a sizable fraction of the inputs.  But in fact there are ways to use dynamic programming more aggressively to solve this problem using much less memory.  There are ways to solve this problem that only require storing the **lengths** of the subsequences, rather than the subsequences themselves, and there are ways to "throw away" some of the stored results as you no longer need them.  That would eliminate a lot of work -- constructing the subsequence arrays isn't cheap.

However, I don't want to get into that level of detail on this particular problem.  Clearly, dynamic programming is a huge topic, and this just gives a brief taste. My point is simply that **caching the solutions of problems you've already solved gives a potentially huge performance win, particularly when doing recursive calculations.  **

