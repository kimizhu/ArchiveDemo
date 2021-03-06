<div id="page">

# Turning a bug into a feature

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2010 6:54:00 AM

-----

<div id="content">

<div class="mine">

I was amused to read [this post about an arithmetic bug which accidentally turned into an AI feature](http://blogs.msdn.com/shawnhar/archive/2009/12/29/bug-or-feature.aspx) (found via [Raymond’s recent link clearance](http://blogs.msdn.com/oldnewthing/archive/2010/03/31/9987780.aspx).) It reminded me of a story that my friend Lars told me about working on a certain well-known first-person shooter back in the day.

The developers of this game had a problem: an enemy computer-controlled character would see a fatal threat (say, a thrown grenade) and run away from it. Trouble is, the half-dozen other enemies in the area would see the same threat and they would all try to leave *via the same best exit path*. They’d bump into each other, shuffle around, reverse direction… and the result was an unrealistic-looking mess that called attention to the artificiality of the world. Basically, the AI had a bug; it was not smart enough to find efficient paths for everyone, and was thereby making the game less fun.

You might try to solve this problem by implementing a more complex and nuanced escape route finding algorithm for cases where multiple AI characters are all faced with threats. However, machine cycles are a scarce resource in first-person shooters; this solution probably doesn’t fit into the performance budget and it’s a lot of dev work. They finally solved the problem by writing a cheap “reverse detector” that detected when an AI character had radically changed direction more than once in a short time period. When the detector notices that an AI character has been running in two different directions in quick succession, the character’s default behavior is changed to some variation on “crouch down and cover your head”. With this policy in place an enemy might run away from your grenade, bump into another fleeing enemy, turn back to try to find a new route, and that would trigger the duck-and-cover behaviour. The net result is that not only does this look like realistic human behavior, it is very satisfying to the player who threw the grenade. I also noticed that Shawn’s post about the arithmetic bug was part of a feature whereby the AI of the racing game was tuned not to make the AI strive to *win the game*, but rather to make the AI strive to *produce a more enjoyable playing experience for the human player*. Lars wrote [a short paper on the subject of “artificial stupidity”](http://lars.liden.cc/Publications/Downloads/2003_AIWisdom.pdf) – that is, deliberately designing flaws into AI systems so that they *appear* intelligent while at the same time creating fun situations rather than frustrating situations. Quite an interesting read. (See [this book](http://www.amazon.com/gp/product/1584502894) for more thoughts on game AI design.)

Sadly, bugs in *compilers* seldom turn out to actually be desirable features, though it does occasionally happen.

</div>

</div>

</div>

