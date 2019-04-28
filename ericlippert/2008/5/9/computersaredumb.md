# Computers are dumb

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/9/2008 11:20:00 AM

-----

 

A few short takes today, from questions I've received recently about LINQ in C\# 3.0.

The first question was "*in the following code, does it really check every single non-negative integer, or does it use the knowledge that once you're beyond ten, you can stop iterating?*"

 

var smallNumbers = Enumerable.Range(0, int.MaxValue).Where(n =\> n \< 10);  
foreach (int i in smallNumbers)  
    Console.WriteLine(i);  

  
The former. You asked LINQ to Objects to apply a predicate to a sequence and that's what it does. If the sequence has two billion elements in it, it runs the predicate two billion times.

You certainly could build your own range object which had a Where method that took an expression tree. Your Where method could then analyze the range and the expression tree in an attempt to build a new range object that iterated a smaller range. Analyzing simple predicates would be pretty easy; more complex predicates would have to be turned back into delegates.

Of course, if it hurts when you do that, *don't do that*. if you already know the range you want, just restrict it in the range constructor in the first place\!

The second question was "*which is the preferred coding style?*" :

 

if (0 \!= customers.Count()) ...

vs

 

if (customers.Any()) ...

On efficiency grounds alone the latter is far preferable. Suppose you have a box which may contain between zero and two billion pennies. You want to take some action if there are any pennies in the box, but you don't care how many there are, only whether there are zero or more. In that scenario, clearly it makes no sense whatsoever to count them all. The Count method doesn't know that all you care about is whether it's going to return zero or not. You asked it to count, so it counts.

(I am reminded of an old joke: on a bank teller's first day, he is asked to count a stack of twenties and verify that there are one hundred in the stack. He starts counting them from one pile into another: one, two, three, ... but when he gets to fifty, he stops and says to his boss "I'll stop here -- if it's right all the way up to fifty, odds are good it's right the whole way.")

Even leaving the obvious efficiency concern aside, the latter code reads better. It more clearly reflects the intention of the programmer.

The third question was "*when iterating through a collection, I noticed that the former code is much faster than the latter code. Any insight as to why?*"

 

myEventLogEntryCollection.CopyTo(myEntriesArray,0);  
for(int i=0; i \< myEntriesArray.Length, i++) {...}

vs

 

foreach(EventLogEntry myEntry in myEventLogEntryCollection) {...}

This made no sense to me. Sure, there will be differences. The former code is using more memory because it makes a copy of the collection; initializing that memory is going to take significant time if the collection is large, but I could see how maybe the array access could be faster than the collection iterator in the loop. Perhaps the improvement in iteration is fast enough that for a large collection, the larger initialization cost was made up. But the user was reporting "*much faster*", not "*slightly faster*". I asked for more details.

What exactly were the timings? *Three seconds vs sixteen seconds.* Holy goodness\! I had figured we were talking about *microseconds* of difference here, not actual humans sitting around watching the clock time. What on earth was the user doing? *Accessing event logs from a remote machine over a slow network.*

Aha\! Now it is clear what is going on here. The iteration time is being dominated by the round trip to the remote machine; the former code makes one very expensive remote call that takes three seconds. The latter code makes hundreds of cheaper remote calls, possibly several per loop iteration, that take a fraction of a second each but which add up to more expense in the long run.

My "insights" were therefore:

  - **You can sometimes trade increased memory use for decreased time. **
  - **The cost of fetching n items by making m requests might be dominated by m, not n.**
  - **If you don't actually describe the relevant features of the problem, it is impossible to get a correct analysis.**

I blogged about these three questions not because I got them all in the same day, but because there is a common thread to the answers: **computers are dumb**. They do exactly what you tell them. If you want performance savings by eliminating unnecessary work through logical deductions about what work can be eliminated, you're going to have to be the one to do make those deductions\!

