<div id="page">

# 250% of what, exactly?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/1/2005 1:13:00 PM

-----

<div id="content">

I just got a question this morning about how to take two collections of items and determine how many of those items had the same name. The user had written this straightforward but extremely slow VBScript algorithm: For Each Frog In Frogs  
  For Each Toad In Toads  
    If Frog.Name = Toad.Name Then  
      SameName = SameName + 1  
      Exit For  
    End If  
  Next  
Next There were about 5000 frogs, 3000 toads and 1500 of them had the same name. Every one of the 3500 unsuccessful searches checked all 3000 toads, and the 1500 successful searches on average checked 1500 toads each. Each time through the inner loop does one loop iteration, two calls to the Name property, one string comparison. Add all those up and you get roughly 50 million function calls to determine this count. This code has been somewhat simplified – the actual user code was doing more work inside the loop, including calls to WMI objects. The whole thing was taking 10+ minutes, which is actually pretty fast considering how much work was being done. Each individual function call was only taking a few microseconds, but fifty million calls adds up\! Now, we've been down this road before in this blog ( [<span class="underline">here</span>](http://blogs.msdn.com/ericlippert/archive/2004/05/12/130840.aspx), [<span class="underline">here</span>](http://blogs.msdn.com/ericlippert/archive/2004/05/13/131533.aspx) and [<span class="underline">here</span>](http://blogs.msdn.com/ericlippert/archive/2004/05/14/132160.aspx)) and so of course I recommended building a faster lookup table rather than doing a full search through the collection every time. Set FrogLookup = CreateObject("Scripting.Dictionary")  
For Each Frog In Frogs  
  FrogLookup(Frog.Name) = Frog  
Next  
For Each Toad In Toads  
  If FrogLookup.Exists(Toad.Name) Then SameName = SameName + 1  
Next Which is much, much faster. That's only about 16 thousand function calls. Now, this is maybe not an apples-to-apples comparison of function calls, but we at least have good reason to believe that this is going to be several times faster.  And indeed it was. But I'm still not sure how much, which brings me to the *actual subject* of today's blog. The user reported that the new algorithm was "250% better". I hear results like this all the time, and I always have to ask to clarify what it means. You can't just say "n% better" without somehow also communicating what you're measuring. (UPDATE: This reported result understandably confused some readers.  Clearly the new loop here is *thousands* of times faster, not a mere 250% faster. As I said before, the sketch above is highly simplified code. The real-world code included many thousands of additional calls to WMI objects which were not eliminated by this optimization. Eliminating these 50 million function calls helped -- you should always eliminate the slowest thing first\!  But doing so also exposed a new "hot spot" that needed further optimization.  However, the point of this article is not the benefits of using lookup tables, but rather that using unexplained percentages to report performance results is potentially misleading.  The result above is merely illustrative.  See the comments for more details.) Allow me to illustrate. Suppose I have a web server that is serving up ten thousand pages every ten seconds. I make a performance improvement to the page generator so that it is now serving up fifteen thousand pages every ten seconds. I can sensibly say:

  - performance has improved by 50%, because we are now serving up 5000 more pages every ten seconds, and 5000 is 50% of the original 10000.  In this world, any positive percentage is good.
  - performance is now 150% of original performance because 15000 is 150% of 10000.  In this world, 0%-100% worse or the same, 100%+ is good.
  - We've gone from 1000 microseconds per page to 667 microseconds per page, saving 333 microseconds per page. 333 is 33% of 1000, so we've got a 33% performance improvement.  In this world, 0% is bad, 100% is perfect,  more than 100% is nonsensical.

If I can sensibly say that performance is better by 50%, 150% and 33%, all referring to exactly the same improvement, then I cannot actually be communicating any fact to my listeners\! They need to know **whether I'm measuring speed or time**, and if speed, whether I'm comparing **original speed to new speed** or original **speed to the difference.** So what is "250%" better? Clearly not raw timings. If this loop took 10 minutes to run before, 250% of 10 minutes is 25 minutes, but surely it is not running in -15 minutes now\! I assume that the measurement is speed -- say, number of loops runnable per hour. If before it took ten minutes and therefore we could run this loop six times per hour, and now it is 250% better, 250% of 6 is 15, and this is "better", so we need to add. So that's 21 loops per hour, or about 3 minutes per loop. Had the user accidentally said "250% faster" but meant to say "250% of previous performance" then we'd conclude that 250% of 6 per hour is 15 per hour, so now we've got 4 minutes per loop. And of course, this is assuming that the person reporting the improvement is actually trying to communicate facts to the audience. Be wary\! Since 33%, 50% and 150% are all sensible ways to report this improvement, which do you think someone who wants to impress you is going to choose? There are opportunities here for what Kee Dewdney calls "percentage pumping" and other shyster tricks for making weak numbers look more impressive. Professor Dewdney's book on the subject, "200% of Nothing", is quite entertaining. The moral of the story is: **please do not report performance improvements in percentages**. Report performance improvements in terms of **raw numbers with units attached**, such as "microseconds per call, before and after change", or "pages per second, before and after change". That gives the reader enough information to actually understand the result. (And the other moral is, of course, **lookup tables are your friends**.)

</div>

</div>

