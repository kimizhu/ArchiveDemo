# Whidbey Island And Bagel Mathematics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/15/2004 5:55:00 AM

-----

[I was highly amused to read on Raymond Chen's blog the other day that mathematicians are hard at work solving the problem of how to most evenly distribute poppyseeds over a bagel](http://blogs.msdn.com/oldnewthing/archive/2004/12/14/300205.aspx). The reason I was highly amused was not just the whimsical description of what is actually a quite practical and difficult problem.

And yes, believe it or not, it is a practical problem; if you can figure out how to distribute points evenly around an arbitrary shape then you can use that information to develop more efficient computer algorithms to solve complex calculus problems that come up in physics all the time. There are also applications in computer graphics, I'd imagine; 3-D modeling frequently requires generating well-behaved finite approximations of a surface.

But I digress. The other reason I was highly amused is that Whidbey Island is the longest island in the United States.

OK, obviously I should explain that one, as the connection between mathematicians, bagels and Whidbey Island may not be immediately apparent.  But first, yes, I know that Whidbey Island is NOT the longest island in the United States -- not that anyone here in Washington State will admit to it.  [As this page points out, by any reasonable measurement Whidbey Island is shorter than Long Island, Hawaii and almost two dozen islands off the coast of Alaska.](http://www.peakbagger.com/pbgeog/longisl.aspx)  Whidbey Island's real claim to fame is that it is the longest island in the United States that is also in the Pacific Ocean but not part of Alaska or Hawaii -- big whoop.

Anyway, when I first moved here I did a bicycle trip around the southern lobe of Whidbey Island, and plenty of people told me lies about Whidbey Island being the longest island in the United States.  Which got me wondering how exactly that would be measured.

This led to a spirited email debate amongst my various math geek friends. The page I referenced above gives three definitions for island length, all vague and imprecise:

  - if the island is "narrow and straight" then "it's easy" to determine what the length is.
  - if the island is "rounded" then the longest distance to "opposite shores" is the length.
  - if the island is "bent", like Whidbey, then draw a bent line "down the middle" and measure that.

I didn't like any of these then, and I don't like them now.  There should be one unambiguous standard for length that doesn't rely upon vague and imprecise notions like "opposite shores", or "down the middle".

The standard should also work for *really* weird-shaped islands.  Consider for example an island which is a narrow ring.  Say, a circle with a circular lake in the middle.  None of the algorithms above are satisfactory. That island should be *longer* than an island with the same size and shape but no lake in the middle. In the island with the lake in the middle you can't walk in a straight line from one side to the other, so the island should be longer. Similarly, islands like Whidbey that are bent should be longer than straight islands.  Or, put another way, taking a long, straight island and bending it into a spiral should not change its length drastically.

Here's how I define the length of an island:  consider every pair of points on the perimeter of the island.  Clearly there are an infinite number of pairs of points, but we won't let that deter us.  Pick a pair.  Now consider every possible path, no matter how twisty-turny, that connects those two points, with the restriction that the path must be entirely on the island at all times.  Clearly, there's an infinite number of those too.

But of that infinite number of paths between the two points, at least one of them has to be the shortest.  (There could be two or more equal shortest paths, if, say, you could go both ways around an internal lake to get from point A to B, but whatever, pick one of them.)  Of course, for most of them it will be a straight line, but on weird shaped islands, it might have to bend here and there.

Now, find the pair of points where the *shortest* distance between them is *as long as possible*.  That is, find the two points on the shore where, even if you took the *shortest* path between those points, you'd still have to walk *farther* than if you took the shortest path between any other pair of points.

That distance, **the length of the longest shortest path,** is the length of the island. 

Of course, finding those points and the shortest path between them might be a difficult problem if the island is a really strange shape.  But that's just a technical detail that we can wave away; we have an impractical but formal definition, and that's what matters to me when I'm wearing my mathematician hat.

What does this have to do with the bagel algorithm?

Imagine all the possible distributions of, say, a thousand points around a "manifold".  (Which is just a fancy way of saying "shape" for our purposes; a more technical definition would take too long.)  Suppose the manifold is the surface of the earth.  Obviously there are an infinite number of such distributions, and an infinite number of them are going to be really, really sucky from the perspective of spreading them around evenly. 

Like, any configurations in which all of the points are on Whidbey Island, for example. 

For each of those distributions of a thousand points, pick every pair, and again, consider every path between each pair, where the path stays on the manifold. For every pair, find the *shortest* path, and then find the pair that has the *shortest shortest* path. 

If all the points in a given configuration are in Whidbey Island, maybe the shortest shortest path is one centimeter, maybe it's one meter, but you are guaranteed that the shortest shortest path is quite a bit smaller than the length of the island\!

Now work out the shortest shortest path for *every possible configuration of points*.  At least one of those configurations has got to have the *longest* shortest shortest path, and that configuration is the desired configuration.  It's the configuration of points where the smallest possible distance between the two points that are closest together cannot be made larger without making some other pair of points become even closer together than the original pair.  You simply can't get any better than the configuration with the longest shortest shortest path. 

Of course, finding that configuration out of the infinite number of possibilities is a non-trivial problem. But clearly the "find the length of the island" problem is just a very simple version of the bagel problem.  It's got only two points and a Whidbey-Island shaped manifold, rather than a thousand points and a bagel-shaped manifold. If you can solve one, you can solve the other\!

And while we're on the subject of island superlatives and ridiculous phrases like "the longest shortest shortest path", I should point out that Canada is home to not only the largest island in a freshwater lake in the world, but also home to the largest island in a lake that is on an island in a lake\! Anyone care to hazard a guess at what they are?

