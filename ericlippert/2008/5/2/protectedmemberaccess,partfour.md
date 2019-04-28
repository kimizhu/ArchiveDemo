# Protected Member Access, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/2/2008 12:13:50 PM

-----

In [Part Two](http://blogs.msdn.com/ericlippert/archive/2008/03/28/why-can-t-i-access-a-protected-member-from-a-derived-class-part-two-why-can-i.aspx) I asked a couple of follow-up questions, the first of which was:

> Suppose you were a hostile third party and you wanted to mess up the parenting invariant. Clearly, if you are sufficiently trusted, you can always use private reflection or unsafe code to muck around with the state directly, so that's not a very interesting attack. Any other bright ideas come to mind for ways that this code is vulnerable to tampering?

Before I get into some ideas for attacks, I want to re-emphasize the bit in the middle there: **attacks which presuppose that the attacker is already an administrator on your machine are not interesting attacks**. A number of people came up with attacks involving the attacker being able to run debuggers or even replace the entire CLR with a custom-built runtime.

Since there is no point in protecting code from hostile code which is *already stronger than the security system itself*, it's not very interesting to consider such attacks. The interesting question is how to protect yourself against hostile code which is *weaker* than the code you are attempting to protect. That's what attackers want to do -- they want to leverage vulnerabilities in your strong code to turn their weak code into strong code. If they already have strong code, they don't need to lure your code into doing something, they can just do it directly.

Anyway, that said, there are a number of flaws in my proposed implementation from a security perspective:

  - Eamon Nerbonne pointed out the implementation is not in the slightest thread-safe; hostile code which wished to mess up the parenting invariant could run adds and removes on the same objects on multiple threads at once, and when the smoke clears, pretty much any weird parenting arrangement you care to name could be the case.
  - Matt pointed out that the implementation relies upon a correct implementation of hashing/equality. If a hostile or buggy object provides an implementation of hashing which is inconsistent with its implementation of equality, it is possible to put stuff into a hash set and never get it back out again. The parenting invariant in this implementation relies upon being able to use hash sets correctly.
  - Jon Skeet pointed out that the implementation has no protections against parenting relationships which are locally sensible but not globally sensible, like a box containing itself.

All of these vulnerabilities could of course be mitigated without too much difficulty; the trick is in remembering to look for the vulnerability in the first place\!

Getting the parenting relationship consistently acyclic is an interesting algorithmic problem. There are numerous ways to do it. Two sketches for your consideration:

1\) Every time you add an item to a container, search the parent list of the container for the item. Hopefully containment relationships are shallow and this is not too expensive. 

2\) Only allow adding an item to a container if the item and the container are different but are themselves in the same immediate container. If you have a box in a bag, and you want to put a ball in the box, first put it in the bag, and then in the box. 

Next time, some thoughts on immutability and parenting relationships.

