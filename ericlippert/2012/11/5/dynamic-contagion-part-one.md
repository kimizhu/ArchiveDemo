<div id="page">

# Dynamic contagion, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/5/2012 6:47:00 AM

-----

<div id="content">

<div class="mine">

[![biohazard](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/5684.biohazard_thumb.gif "biohazard")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/5661.biohazard_2.gif)Suppose you're an epidemiologist modeling the potential spread of a highly infectious disease. The straightforward way to model such a series of unfortunate events is to assume that the population can be divided into three sets: the definitely infected, the definitely healthy, and the possibly infected. If a member of the healthy population encounters a member of the definitely infected or possibly infected population, then they become a member of the possibly infected population. (Or, put another way, the possibly infected population is closed transitively over the exposure relation.) A member of the possibly infected population becomes classified as either definitely healthy or definitely infected when they undergo some sort of test. And an infected person can become a healthy person by being cured.

This sort of contagion model is fairly common in the design of computer systems. For example, suppose you have a web site that takes in strings from users, stores them in a database, and serves them up to other users. Like, say, this blog, which takes in comments from you, stores them in a database, and then serves them right back up to other users. That's a [Cross Site Scripting](http://en.wikipedia.org/wiki/Cross-site_scripting) (XSS) attack waiting to happen right there. A common way to mitigate the XSS problem is to use [data tainting](http://en.wikipedia.org/wiki/Taint_checking), which uses the contagion model to identify strings that are possibly hostile. Whenever you do anything to a potentially-hostile string, like, say, concatenate it with a non-hostile string, the result is a possibly-hostile string. If the string is determined via some test to be benign, or can have its potentially hostile parts stripped out, then it becomes safe.

The "dynamic" feature in C\# 4 and above has a lot in common with these sorts of contagion models. As I pointed out last time, when an argument of a call is dynamic then odds are pretty good that the compiler will classify the result of the call as dynamic as well; the taint spreads. In fact, when you use almost any operator on a dynamic expression, the result is of dynamic type, with a few exceptions. ("is" for example always returns a bool.)  You can "cure" an expression to prevent it spreading dynamicism by casting it to object, or to whatever other non-dynamic type you'd like; casting dynamic to object is an identity conversion.

The way that dynamic is contagious is an emergent phenomenon of the rules for working out the types of expressions in C\#. There is, however, one place where we explicitly use a contagion model inside the compiler in order to correctly work out the type of an expression that involves dynamic types: it is one of the most arcane aspects of method type inference. Next time I'll give you all the rundown on that.

</div>

</div>

</div>

