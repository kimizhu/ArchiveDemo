# What To Do When The Source Control Server Is Down

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/30/2008 1:01:00 PM

-----

I have not forgotten about my series on method type inference; rather, the contrary. I have been thinking hard about how to change method type inference to be more accurate in a hypothetical world with covariant and contravariant interfaces, and this has led me to dig in even deeper to the method type inference specification and implementation. I've got acres of notes on this now; getting them into bloggable form will take more time.

Until then, some of the wit and wisdom of the managed languages team.

 We "dogfood" our source control system here; that is, we now use the same source control system that we sell to customers to control our own sources. Even better, we use recent builds of the source control system, so that we find the flaws in new versions of it before customers do. We feel the pain so that you don't have to. But this means that occasionally the [Software Development Company Abstraction](http://www.joelonsoftware.com/articles/DevelopmentAbstraction.html) breaks down for a few hours here and there.

Some wags have started writing suggestions for what to do when the source control server is down on the whiteboard outside my office. So far, the list consists of:

0\) Teach the VB team what index lists begin at.  
1\) Send email to \[the source control server team\].  
2\) Complain.  
3\) Update this whiteboard.  
4\) [Learn helplessness](http://en.wikipedia.org/wiki/Learned_helplessness).  
5\) Go to [Tosche Station](http://starwars.wikia.com/wiki/Tosche_Station). Pick up some power converters.  
6\) Consider the management track.  
7\) Write a new source control system -- yes, you have enough time.

What do you do when your source control / bug database / network / email / whatever crucial system is temporarily unavailable?

