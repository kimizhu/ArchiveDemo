# Santalic tailfans, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/6/2009 1:19:31 PM

-----

As I have said before [many times](http://blogs.msdn.com/ericlippert/archive/tags/Performance/default.aspx), there is [only one sensible way](http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx) to make a performant application. (As an aside: perfectly good word, performant, deal with it\!) That is:

  - Set meaningful, measurable, customer-focused goals.
  - Write the code to be as clear and correct as possible.
  - Carefully measure your performance against your goals.
  - Did you meet your goal? Great\! Don't waste any time on performance analysis. Spend your valuable time on features, documentation, bug fixing, robustness, security, whatever.
  - If you did not meet your goal, use tools to discover what the worst-performing fixable thing is, and fix it.

Iterate on that process, updating your goals if it proves necessary, until you either have something that meets your goals or you give up.

My explicit goal for my [little dictionary search program](http://blogs.msdn.com/ericlippert/archive/2009/02/04/a-nasality-talisman-for-the-sultana-analyst.aspx) was that it give results fast enough that I would not be sitting there impatiently waiting for more than a few seconds. That's a very customer-focused goal, me being the only customer of this program. With only a very small amount of tweaking my program met that goal right away, so why would I spend any more time on it? The program takes just under two seconds to report the results for a typical query, which is faster than I can read.

But suppose that we did want to improve the performance of this program for some reason. How? Well, let's go down the list. We have goals, we have very clear code, we can measure the performance easily enough. Suppose we didn't meet the goal. The last thing on the list is "use tools to discover what the slowest fixable thing is".

A commenter conjectured that the performance bottleneck of this program was in the disk I/O. As you can see, every time we do a query we re-read the two megabyte dictionary file dozens of times. This has the benefit of using very little memory; we never have more than one line of the dictionary in memory at a time, instead of the whole four megs it would take to store the dictionary (remember, the dictionary is in ASCII on disk but two-byte Unicode if in strings in memory, so the in-memory size will double.)

That's a conjecture -- a reasonable conjecture -- but nevertheless, it's just a guess. If I've learned one thing about performance analysis it's that my guesses about where the bottleneck is are often wrong. I'll tell you right now that yes, the disk-hitting performance is bad, but it is not the worst thing in this program, not by far. **Where's the real performance bottleneck? Any guesses? Could you know without using tools?**

Here's the result of a timing profile run of a seven-letter queries with one blank, repeated 20 times:

**43%: Contains  
**21%: ReadLine  
14%: Sort  
7%: ToUpperInvariant  
15%: everything else

Holy goodness\! The call to the Contains extension method in the query to test whether the dictionary word is in the rack set array is almost half the cost of this program\!

Which makes sense, once you stop to think about it. The "Contains" method is by its highly general nature necessarily very naive. When given an array, 99.9% of the time it has to look through the entire 26-item array because 99.9% of the time, the word is not actually going to match any of the possible racks. It cannot take advantage of any "early outs" like you could do if you were doing a linear search on a sorted list. And each time through the array it has to do a full-on string comparison; there's no fancy-pants checks in there that take advantage of string immutability or hash codes or any such thing.

We have a data structure that is designed to rapidly tell you whether a member is contained in a set. And even better, it already does the "distinct" logic. When I replace

 

    var racks = (from rack in ReplaceQuestionMarks(originalRack)  
                 select Canonicalize(rack)).Distinct().ToArray();

with

 

    var racks = new HashSet\<string\>(  
                from rack in ReplaceQuestionMarks(originalRack)  
                select Canonicalize(rack));

suddenly performance improves massively. The "Contains" drops down to 3% of the total cost, and of course, the total cost is now half of what it was before.

Another subtle point here: notice how when I changed the type of the variable "racks" from "array of string" to "set of string", I didn't have to redundantly change the type thanks to implicit typing of local variables. I want to emphasize the semantics here, not the storage mechanism. If I felt that communicating the storage mechanism was an important part of this code -- because it has such a strong effect on performance -- perhaps I would choose to emphasize the storage by eschewing the "var".

With this change, the program performance improves to about one second per query and the profile now looks like this:

39%: ReadLine  
23%: Sort  
11%: ToUpperInvariant  
7%: the iterator block goo in FileLines  
5%: ToArray (called by Join)  
15%: everything else

Now the bottleneck is clearly the combination of repeated file line reading (48%) and the string canonicalization of every dictionary line over and over again (37%).

With this information, we now have data with which to make sensible investments of time and effort. We could cache portions of the dictionary in memory to avoid the repeated disk cost. Is the potential increase in speed worth the potentially massive increase in memory usage? We could be smart about it and, say, only cache the seven- and eight-letter words in memory.

We could also attack the canonicalization performance problem. Should we perhaps precompute an index for the dictionary where every string is already in canonical form? This in effect trades increased disk space, increased program complexity and increased redundancy for decreased time. Should we use a different canonicalization algorithm entirely?

All of these decisions are driven by the fact that I have already exceeded my performance goal, so the answer is "no". Good enough is, by definition, good enough. If I were using this algorithm to actually build a game AI, it would not be good enough anymore and I'd go with some more clever solution. But I'm not.

