# Long division

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/28/2009 1:10:56 PM

-----

A thing that makes a reader go hmmm is why in C\#, int divided by long has a result of long, even though it is clear that when an int is divided by a (nonzero) long, the result always fits into an int.

I agree that this is a bit of a head scratcher. After scratching my head for a while, two reasons to not have the proposed behaviour came to mind.

First, why is it even desirable to have the result fit into an int? You'd be saving merely four bytes of memory and probably cheap stack memory at that. You're already doing the math in longs anyway; it would probably be more expensive in time to truncate the result down to int. Let's not have a false economy here. Bytes are cheap.

A second and more important reason is illustrated by this case:

 

long x = whatever;  
x = 2000000000 + 2000000000 / x;

Suppose x is one. Should this be two integers, each equal to two billion, added together with **unchecked integer arithmetic**, resulting in a **negative number** which is then converted to a long? Or should this be the long 4000000000 ?

Once you start doing a calculation in longs, odds are good that it is because you want the range of a long **in the result and in all the calculations that get you to the result**.

