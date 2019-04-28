# Rarefied Heights Part Two, Plus Some Random Short Takes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/2/2004 1:45:00 PM

-----

A few short takes today before I get into the actual subject of today's entry.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Kids Today

My teenaged cousin has apparently accidentally reinvented [pseudolocalization](http://blogs.msdn.com/ericlippert/archive/2004/05/10/129481.aspx).  I was chatting with him the other day and I noticed that his MSN Instant Messenger screen name was ŁıppэяŦ.

D00D, W00T\! Kids today have it so easy. Back in my day we didn't have Unicode; when we wanted to be 3L1T3 on a BBS we had to be content with using numbers in the place of vowels, aggressive use of capitalization, and deliberate mispellings.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Same To You, Buddy

This true story happened oh, about fifteen minutes ago.

I am walking into building 41. A guy is sitting by the front door. I smile and nod as I pass by. The guy says in response "You know what you should do? I'll tell you what you should do\!"

I shoot him an extremely quizzical and rather startled look. Then I realized that he is in fact on a hands free cellphone and only appears to be responding to me.

I'm still not used to modern technology.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

A Natural Theory Of Programming Languages

Stanley Lippman has a wonderful post today that crams the history, theory and practice of language development into a small space. It's called [Towards a Natural Theory of Programming Languages](http://blogs.msdn.com/slippman/archive/2004/06/02/146730.aspx "http://blogs.msdn.com/slippman/archive/2004/06/02/146730.aspx"). The key takeaway is this: **programming languages are tools built at a specific time to solve a specific class of problems on a specific class of hardware**. Many [flame wars](http://www.deftcode.com/archives/every_language_war_ever.html "http://www.deftcode.com/archives/every_language_war_ever.html") could have been ended by both sides recognizing this rather basic fact. I'm looking forward to more posts in this vein, as I think there is a lot more to say on the subject.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Rarefied Heights Revisited

I miss mathematics. My degree was a joint degree in both applied mathematics and computer science, but over the last few years I've gradually been losing the math skills as they've gone unused. I think I might delve into some of the **theoretical-yet-recreational aspects of mathematics in computer science** from time to time. (And just to be clear, by *mathematics* I mean coming up with proofs of theorems, not solving practical problems -- that's *arithmetic*.)

Case in point: a [while back](http://blogs.msdn.com/ericlippert/archive/2004/05/12/130840.aspx "http://blogs.msdn.com/ericlippert/archive/2004/05/12/130840.aspx") I pulled the weak form of **Stirling's Approximation** out of thin air in order to justify the claim that any comparison sort of n elements requires O(n log n) comparisons.

n\! \> (n/e)<sup>n</sup>  

This is *quite* weak.  The strong form of Stirling's Approximation is 

n\! =  √(2nπ) (n/e)<sup>n</sup> (1 + O(1/n)) 

Note the "big O" notation in there.  We're saying that we're not sure exactly how close this approximation is, but that the approximation gets really good *on a percentage basis* for big values of n. 

I feel bad about pulling that result out and using it with no justification whatsoever.  Proving the strong form is kind of a pain in the rear but proving the weak form is really easy.  Since that's all we need, just for the heck of it let's prove the weak form. 

Instead of proving the inequality directly, we'll prove that 

ln n\! \> ln (n/e)<sup>n</sup>  

Which I'm sure you'll agree is basically the same thing -- if one quantity is larger than another, then the log is larger than the log of the other, and vice versa.  

Let's start with the left side.  Clearly 

ln n\! = ln 1 + ln 2 + ln 3 + ... + ln n    

which is a little cumbersome to write, so I'll use big-sigma notation for the sum. 

\= Σln x,   for x from 1 to n. 

This next step is a bit of a hand-wave -- I want to use the fact that this sum is the area of a "stair step" function, and that the area of a stair-step function is always larger than the area under a function which it dominates.  Rather than proving that formally, I'll give a graphical justification. 

Let's consider n = 4.  I've graphed out ln x, and I'm sure that you will agree that this sum is equal to the total area bounded by the three red boxes.  (ln 1 is zero, so we can ignore it -- it's "red box" would have zero height and hence zero area.)  

![Log graph](http://www.eric.lippert.com/log.jpg "Log graph")

Let's call the area computed by adding together a bunch of rectangles the "quadrature".  Clearly the quadrature is larger than the (blue) area under the curve not just for n = 4 but for *all* n.  We can compute the blue area by integrating, so let's declare this inequality. 

Σln x  \>= ∫ln x dx,   for x from 1 to n. 

The antiderivative of ln x is x ln x - x, so we have 

ln n\!  
\= Σln x   
\>= ∫ln x dx  
\= (n ln n - n) - (1 ln 1 - 1)  
\= n ln n - n + 1  
\> n ln n - n  
\= ln (n/e)<sup>n</sup>  

And we're done\!  The weak form of Stirling's approximation is justified.  

Didn't that feel good?  How often do you get to use calculus to prove a fact about the performance characteristics of sorting lists?

