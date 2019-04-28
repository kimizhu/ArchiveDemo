<div id="page">

# Dude, Where's My Asymptotic Order Analysis?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/14/2004 12:12:00 PM

-----

<div id="content">

<span>"There is something missing from this analysis.  Saying that this is an </span><span>O(n) </span><span>algorithm perhaps isn't quite the whole story.  What did I leave out?</span><span> </span> <span></span> <span>That line in [yesterday's post ](http://blogs.msdn.com/ericlippert/archive/2004/05/13/131533.aspx)generated lots of good commentary.  Several people mentioned some interesting ideas that I wasn't expecting, and no one mentioned the thing that I was thinking was the obvious flaw in my analysis.  (I guess my skills at misdirection are better than I thought. </span><span>J</span><span> )  First, some of the interesting things people pointed out: </span>

<span></span> <span>Mike Pope pointed out that in addition to the generics, which I called out as "not available until Whidbey", **<span>the Boolean on </span>**</span>**<span>Split</span><span> is also not available yet</span>**<span>.  I hadn't realized that; I've been using the Whidbey framework for so long now that I sometimes get confused over what is a new feature and what isn't.  Sorry about that. </span>

<span></span> <span>Jason pointed out that **<span>my analysis completely ignored the question of memory usage</span>**.  This algorithm is a huge memory hog.  First it copies the entire original string into an array, then it copies the array into two hash tables which, if there are a lot of unique words, are a significant fraction of the original string. If the data are big then the working set can get really, really big.  </span> <span>If memory consumption were a problem, one could rewrite the algorithm to lessen the impact.  For instance, we could tokenize on the fly, one line at a time, rather than reading the whole file in and tokenizing it all at once.  And we could dispose of everything as soon as we're done using it.  But in general, this thing consumes <span>O(n)</span> **** extra memory.  There are sort algorithms which consume <span>O(log n)</span> or <span>O(1)</span>extra memory. </span>

<span></span> <span>Raymond Chen pointed out that <span>**assuming that hash table inserts are** <span>O(1)</span> **is an optimistic assumption**</span>, and that for most hash table algorithms **<span>there are worst-cases that make for an </span>**</span><span>O(n) </span>**<span>experience</span>**<span>.  Indeed, I am making the assumption that the word list I'm parsing has not been specifically chosen by a hostile entity in order to gum up the works of my hash table\!  Designing a hash table that gives </span><span>O(1) </span><span>performance even in the face of hostile data is a hard problem. </span>

<span></span> <span>Mike Dimmick conjectured correctly that the **<span>actual</span>** **<span>implementation of the generic dictionary is an extensible-hashing-with-chaining algorithm that re-hashes when the table gets too full</span>**.  This ensures that the hash chains are always of reasonable length on average (assuming non-hostile data\!)  Re-hashing is expensive, and we could optimize this algorithm further by hinting to the table what its maximum number of entries is going to be.  </span> <span>These are all good points, Mike, but re-hashing is never more expensive than </span><span>O(n).</span><span> T</span><span>he **<span>amortized</span>** cost of re-hashing is </span><span>O(1) </span><span>per insert.  (Suppose we have 32 items, and we re-hash 2 items when there are 2 items, 4 when there are 4, 8 when there are 8, and so on.  We end up re-hashing 63 items altogether.  No matter how big the table gets, the number of items re-hashed so far is always less than twice the table size.) </span>

<span></span> <span>Mike also wonders **<span>how expensive it is for the table to determine the size of the internal bucket array, which must be a prime number</span>**.  The hash algorithm has precalculated prime bucket sizes ranging from 3 to 7199369.  If you have more than seven million items then it gets a little weird because we need to find a prime number larger than </span><span>n </span><span>the hard way, by testing each candidate number with all its divisors.  Figuring out the average, amortized and worst-case performance of a naïve prime finder is an interesting topic which I might go into more detail on later.  </span>

<span></span> <span>Centaur pointed out that **<span>I neglected the real-world constant factor</span>**.  Indeed, it *<span>might</span>* be the case that the overhead of this algorithm is, for **<span>typical</span>** cases, larger than the cost of an </span><span>O(n log n) </span><span>algorithm, and hence the asymptotically slower algorithm would be preferred.  Armchair perf analysis must be accompanied by real-world analysis\! </span>

<span></span> <span>David Thomas did a quick real-world analysis on a hash table of integers (nice\!) and found that indeed, **<span>the cost of inserts does rise as the hash table gets very large</span>**.  I don't know about the .NET Framework, but I do know that in the JScript engine, the cost of hash table inserts becomes large not because of the hash table, but because**<span> as the working set grows, the garbage collector takes longer and longer to run</span>**.  The hash table stays extremely efficient, but the infrastructure around it gets slower. This is another practical factor which I neglected to call out.  Actually determining whether the GC was the culprit in David's benchmark would require deeper analysis.  (It would also be interesting to see how perf improves by moving from HashTable to Dictionary\<int\> -- by eliminating boxing and casting, the perf ought to improve considerably.) </span>

<span></span> <span>Those are all good points.  So what was I thinking of that no one mentioned?  I hand-waved away important stuff at least twice. </span>

<span></span> <span class="underline"><span>Handwave Numero Uno </span></span>

<span></span>

> <span>The internals of the table lookups do data comparisons to determine **<span>equality</span>**, but not **<span>ordering</span>**, … so they don't count. </span><span></span>

<span></span> <span>Oh really?  I'm assuming that comparing two strings for equality is an </span><span>O(1) </span><span>operation.  What if the document consists of very, very long strings that have big common prefixes?  For example, what if the list of strings is </span>

<span></span> <span>banananananana...narama  
</span><span>banananananana...naramas  
</span><span>banananananana...naramas </span>

<span></span> <span>and so on?  I have not taken into account the fact that determining whether two strings really are "the same" or not is expensive if they are **<span>big and similar at the beginning</span>**.  Now, for my example of Google queries I know that the strings are going to be short (and non-hostile\!) so I feel good about saying that hash table operations on them are </span><span>O(1). </span><span> A better theoretical treatment of this algorithm would call out that assumption. </span>

<span></span> <span class="underline"><span>Handwave Numero Two-o </span></span>

<span></span>

> <span>Suppose that there are </span><span>n</span><span> words in the list obtained from \[subproblem\] (1) \[Obtain a list of words.\]  ... <span>We've solved subproblem (1).  </span></span>

<span></span> <span>Dudes, **<span>where's the order analysis </span>**for that step?  I completely blew past it without even talking about it\! </span>

<span></span> <span>Tokenizing an arbitrary string with a list of arbitrary separators is not necessarily cheap\!  I spend my whole time talking about <span>n</span>, the number of words, but never talk about the size of the file that contains that word list\!  </span> <span>I just checked the source code, and the tokenizer that takes an array of </span><span>c </span><span>char delimiters and a source string of </span><span>s</span><span> characters uses an </span><span>O(cs) </span><span>algorithm.  There are  </span><span>O(c+s) </span><span> algorithms for that, but then you have to consider the stuff that Centaur pointed out:  in **<span>typical</span>** cases, the actual cost of the preprocessing necessary to implement the </span><span>O(c+s) </span><span> algorithm might be larger than the cost of running the naïve algorithm.  The naïve algorithm optimizes down to blazingly fast machine code and will therefore outperform the asymptotically faster algorithm in typical small cases.  We usually do not have thousands of delimiters and million-character strings\!  Of course, in our case the number of delimiter characters is a constant, so we can ignore it in the asymptotic analysis.  (If the delimiters are strings, not characters, it gets even more complicated, but we won't go there.) </span>

<span></span> <span>The number of characters in the string is never smaller than the number of words, and often much larger; given this it would be more accurate to say that the whole thing is an </span><span>O(s) </span><span>algorithm, not an </span><span>O(n) </span><span>algorithm.  </span>

<span></span> 

</div>

</div>

