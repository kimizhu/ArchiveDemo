# How Not To Teach Recursion

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/19/2004 6:12:00 PM

-----

A [Joel On Software](http://www.joelonsoftware.com/ "http://www.joelonsoftware.com/") reader asked the other day for examples of recursive functions other than old chestnuts like Fibonacci or factorial.  Excellent question. I suggested [topological sort](http://blogs.msdn.com/ericlippert/archive/2004/03/16/90851.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/16/90851.aspx"), of course, but there are plenty of other examples that are way better than the Fibonacci numbers.  Why do I think Fibonacci is bad?  Read on. 

In case it's slipped your mind, the Fibonacci numbers are defined as follows: 

fib(1) = 1  
fib(2) = 1  
fib(n) = fib(n-1) + fib(n-2) 

thus, they go 1, 1, 2, 3, 5, 8, 13, 21, … 

The Fibonacci numbers have many interesting properties.  For example, suppose you have a pair of rabbits that every year gives birth to a pair of rabbits, rabbits take one year to reach maturity, and rabbits never die.  In that (admittedly unrealistic) situation, the number of pairs alive at any given time is a Fibonacci number.  Clearly the number alive this year is the number alive the year before (who all live) plus the number alive two years before (who are now mature enough to breed.)  Fibonacci numbers also have a relationship to the Golden Mean, logarithmic spirals, and many other interesting nooks of mathematics.  But that's not what I want to talk about today. 

Because they have a straightforward recursive definition, it's natural to introduce recursive programming to students by simply translating the definition into code: 

function fib\_1(n)  
{  
    if (n == 1 || n == 2)  
        return 1;  
    else  
      return fib\_1(n-1) + fib\_1(n-2);  
} 

I am **in principle** very much for this pedagogic approach.  Recursive programming can be difficult to get your head around.  Starting with recursive definitions, getting your head around those, and then using those definitions in code is a pretty good way to approach the material.  If you think of lists as being "zero or more items in a row", it is conceptually hard to think of anything other than an iterative approach to processing the list.  If you think of a list as "a list may be empty, or may be an item followed by a list", then the definition of the data leads itself to a recursive programming approach very naturally. 

**Practically** however, this is probably the worst possible simple way to solve the Fibonacci problem.  When you introduce recursion by using it to solve a problem which iteration solves much, much better, students come away thinking that recursion is dumb -- that it is both harder to understand and produces worse solutions.  Better to pick an example where the iterative solution is not better\! 

In fact, this is a good [whiteboarding question](http://blogs.msdn.com/ericlippert/archive/2004/04/15/114094.aspx)for interviews:  implement me a recursive Fibonacci algorithm, tell me why it is terrible, and write me a better algorithm. A good candidate should be able to come up with the above and something like this: 

function fib\_2(n)  
{  
    if (n == 1 || n == 2)  
        return 1;  
    var fibprev = 1;  
    var fib = 1;  
    for (var cur = 2 ; cur \< n ; ++cur)  
    {  
        var temp = fib;  
        fib += fibprev;  
        fibprev = temp;  
    }  
    return fib;  
} 

Typical questions that I might ask at this point in an interview are  

  - What are the asymptotic running time and memory usage of the two algorithms you've written?
  - Can you write me an algorithm with even better asymptotic running time and memory usage? Alternatively, can you convince me that this is as good as it gets?
  - Suppose I asked you to make this algorithm absolutely as fast as possible.  What would you do?
  - Now suppose that I do not care much about raw speed but I do want the method to be robust in the face of bad input.  Now what do you do?

and so on.  (Anyone who cares to propose answers to these can leave comments; consider this a spoiler warning if you want to work them out for yourself.  Working out the asymptotic running time of recursive fib is quite easy and produces a somewhat surprising result.)

Another old chestnut that's often used as an introduction to recursion is the famous **Tower Of Hanoi** problem.  Briefly stated, you have a number of disks with a hole in the middle, and three spikes upon which you can place disks.  The disks are all of different sizes and are arranged from biggest to smallest in a pyramid on one spike.  You must move one disk at a time from spike to spike, you must never place a larger disk on top of a smaller disk, and you must end up with the entire pyramid on a different spike than you started with.  

The recursive solution is straightforward.  Moving a "sub-pyramid" of size one is obviously trivial.  To move a sub-pyramid of size n, first move a sub-pyramid of size n-1 to an  extra spike.  Then move the bottommost disk of the size n pyramid to the other extra spike.  Then move the sub-pyramid of size n-1 onto the disk on the second extra spike.  A nice recursive solution\! 

Many recursive implementations for this old chestnut exist.  But I'm not a big fan of this problem as a pedagogic approach to recursion for two reasons.  First, it's **totally contrived** (as is “fib“).  Second, there is a very simple iterative solution for this problem, which is almost never mentioned when it is used as an example of the power of recursive programming\!  Again, we should pick examples where recursion is **clearly** preferable. 

In case you're curious, the iterative solution can be produced by writing a program that implements these rules in a loop:  

  - Number the disks 1 to n starting from the top.  
  - Never move an odd disk onto an odd disk.
  - Never move an even disk onto an even disk.
  - Never move a larger disk onto a smaller disk.
  - Never move the same disk twice in a row.
  - Never move a disk onto an empty spike unless you have to.  
  - Move disks until there are no legal moves.

If you don't believe me, try it yourself with ace through ten of a deck of cards sometime.  Or, perhaps start with ace through four -- ace through ten will take you a while.  Both the recursive and iterative solutions are optimal in the number of moves.  The recursive solution is O(n) in terms of memory usage, the iterative solution is O(1).

