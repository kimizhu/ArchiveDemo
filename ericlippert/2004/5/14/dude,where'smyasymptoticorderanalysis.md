# Dude, Where's My Asymptotic Order Analysis?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/14/2004 12:12:00 PM

-----

"There is something missing from this analysis.  Saying that this is an O(n) algorithm perhaps isn't quite the whole story.  What did I leave out?   That line in [yesterday's post ](http://blogs.msdn.com/ericlippert/archive/2004/05/13/131533.aspx)generated lots of good commentary.  Several people mentioned some interesting ideas that I wasn't expecting, and no one mentioned the thing that I was thinking was the obvious flaw in my analysis.  (I guess my skills at misdirection are better than I thought. J )  First, some of the interesting things people pointed out: 

 Mike Pope pointed out that in addition to the generics, which I called out as "not available until Whidbey", **the Boolean on ****Split is also not available yet**.  I hadn't realized that; I've been using the Whidbey framework for so long now that I sometimes get confused over what is a new feature and what isn't.  Sorry about that. 

 Jason pointed out that **my analysis completely ignored the question of memory usage**.  This algorithm is a huge memory hog.  First it copies the entire original string into an array, then it copies the array into two hash tables which, if there are a lot of unique words, are a significant fraction of the original string. If the data are big then the working set can get really, really big.   If memory consumption were a problem, one could rewrite the algorithm to lessen the impact.  For instance, we could tokenize on the fly, one line at a time, rather than reading the whole file in and tokenizing it all at once.  And we could dispose of everything as soon as we're done using it.  But in general, this thing consumes O(n) **** extra memory.  There are sort algorithms which consume O(log n) or O(1)extra memory. 

 Raymond Chen pointed out that **assuming that hash table inserts are** O(1) **is an optimistic assumption**, and that for most hash table algorithms **there are worst-cases that make for an **O(n) **experience**.  Indeed, I am making the assumption that the word list I'm parsing has not been specifically chosen by a hostile entity in order to gum up the works of my hash table\!  Designing a hash table that gives O(1) performance even in the face of hostile data is a hard problem. 

 Mike Dimmick conjectured correctly that the **actual** **implementation of the generic dictionary is an extensible-hashing-with-chaining algorithm that re-hashes when the table gets too full**.  This ensures that the hash chains are always of reasonable length on average (assuming non-hostile data\!)  Re-hashing is expensive, and we could optimize this algorithm further by hinting to the table what its maximum number of entries is going to be.   These are all good points, Mike, but re-hashing is never more expensive than O(n). The **amortized** cost of re-hashing is O(1) per insert.  (Suppose we have 32 items, and we re-hash 2 items when there are 2 items, 4 when there are 4, 8 when there are 8, and so on.  We end up re-hashing 63 items altogether.  No matter how big the table gets, the number of items re-hashed so far is always less than twice the table size.) 

 Mike also wonders **how expensive it is for the table to determine the size of the internal bucket array, which must be a prime number**.  The hash algorithm has precalculated prime bucket sizes ranging from 3 to 7199369.  If you have more than seven million items then it gets a little weird because we need to find a prime number larger than n the hard way, by testing each candidate number with all its divisors.  Figuring out the average, amortized and worst-case performance of a naïve prime finder is an interesting topic which I might go into more detail on later.  

 Centaur pointed out that **I neglected the real-world constant factor**.  Indeed, it *might* be the case that the overhead of this algorithm is, for **typical** cases, larger than the cost of an O(n log n) algorithm, and hence the asymptotically slower algorithm would be preferred.  Armchair perf analysis must be accompanied by real-world analysis\! 

 David Thomas did a quick real-world analysis on a hash table of integers (nice\!) and found that indeed, **the cost of inserts does rise as the hash table gets very large**.  I don't know about the .NET Framework, but I do know that in the JScript engine, the cost of hash table inserts becomes large not because of the hash table, but because** as the working set grows, the garbage collector takes longer and longer to run**.  The hash table stays extremely efficient, but the infrastructure around it gets slower. This is another practical factor which I neglected to call out.  Actually determining whether the GC was the culprit in David's benchmark would require deeper analysis.  (It would also be interesting to see how perf improves by moving from HashTable to Dictionary\<int\> -- by eliminating boxing and casting, the perf ought to improve considerably.) 

 Those are all good points.  So what was I thinking of that no one mentioned?  I hand-waved away important stuff at least twice. 

 Handwave Numero Uno 

> The internals of the table lookups do data comparisons to determine **equality**, but not **ordering**, … so they don't count. 

 Oh really?  I'm assuming that comparing two strings for equality is an O(1) operation.  What if the document consists of very, very long strings that have big common prefixes?  For example, what if the list of strings is 

 banananananana...narama  
banananananana...naramas  
banananananana...naramas 

 and so on?  I have not taken into account the fact that determining whether two strings really are "the same" or not is expensive if they are **big and similar at the beginning**.  Now, for my example of Google queries I know that the strings are going to be short (and non-hostile\!) so I feel good about saying that hash table operations on them are O(1).  A better theoretical treatment of this algorithm would call out that assumption. 

 Handwave Numero Two-o 

> Suppose that there are n words in the list obtained from \[subproblem\] (1) \[Obtain a list of words.\]  ... We've solved subproblem (1).  

 Dudes, **where's the order analysis **for that step?  I completely blew past it without even talking about it\! 

 Tokenizing an arbitrary string with a list of arbitrary separators is not necessarily cheap\!  I spend my whole time talking about n, the number of words, but never talk about the size of the file that contains that word list\!   I just checked the source code, and the tokenizer that takes an array of c char delimiters and a source string of s characters uses an O(cs) algorithm.  There are  O(c+s)  algorithms for that, but then you have to consider the stuff that Centaur pointed out:  in **typical** cases, the actual cost of the preprocessing necessary to implement the O(c+s)  algorithm might be larger than the cost of running the naïve algorithm.  The naïve algorithm optimizes down to blazingly fast machine code and will therefore outperform the asymptotically faster algorithm in typical small cases.  We usually do not have thousands of delimiters and million-character strings\!  Of course, in our case the number of delimiter characters is a constant, so we can ignore it in the asymptotic analysis.  (If the delimiters are strings, not characters, it gets even more complicated, but we won't go there.) 

 The number of characters in the string is never smaller than the number of words, and often much larger; given this it would be more accurate to say that the whole thing is an O(s) algorithm, not an O(n) algorithm.

