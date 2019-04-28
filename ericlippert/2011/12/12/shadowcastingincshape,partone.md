# Shadowcasting in C\#, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/12/2011 7:13:00 AM

-----

I've always loved the "roguelike" games; perhaps you've played some of them. Those are the games where you get a top-down view of a tile-based world, and have as much real time as you like to make a choice of action. The canonical plot is to enter a dungeon, get to the bottom, retrieve the Amulet of Yendor, and make it back out of the dungeon with it. As you might expect, the original game with these characteristics was called "[Rogue](http://roguebasin.roguelikedevelopment.org/index.php/Rogue)", and it has spawned many far more complex imitators. I'm particularly fond of [Nethack](http://nethackwiki.com/wiki/Main_Page), which I have completed twice in many, many hundreds of attempts. Hard game, Nethack.

The original Rogue had a very simplistic approach to lighting the dungeon: when you entered a lit room, you could see everything in the room regardless of whether it was behind an obstacle or not. More modern roguelike games have had increasingly more sophisticated algorithms for determining lighting that take obstacles, light intensity, and so on, into account. I was curious to see what different techniques existed for simulating realistic lighting in roguelike games, and I quickly found [the collection of articles on RogueBasin on "Field of View"](http://roguebasin.roguelikedevelopment.org/index.php/Category:FOV).

Though I really appreciate the effort that went into writing these articles and the implementations -- in particular, the articles by Gordon Lipford, Björn Bergström and Henri Hakl -- I must say that I found a number of them difficult to follow. There are a lot of subtleties to these algorithms, and some of these articles use common important words from geometry (like "slope", "line" and "angle") in unusual and inconsistent ways. When I looked at various implementations of various algorithms that people had - again, very helpfully - published, I found a lot of good stuff but also some questionable programming practices and uncommented subtle choices.

I thought what I might do then, both for my own education and as a public service, is to describe one of the algorithms in *excessive detail*, implement it, and describe the various factors I considered when choosing implementation techniques.

First off, before we get into the details, here's a little Silverlight application that demonstrates what I mean. **Click on the control below** and then use the cursor arrow keys to move you (the "at" sign") around. Notice that you can only see so far -- about nine or ten squares in any direction -- and that obstacles cast shadows. In particular, notice how shadows behave in the region that contains a lot of tightly-spaced "pillars". Do the shadows behave realistically? While pondering that question, see if you can find the treasure and escape with it\!

-----

[![Get Microsoft Silverlight](http://go.microsoft.com/fwlink/?LinkId=161376)](http://go.microsoft.com/fwlink/?LinkID=149156&v=4.0.50826.0)

-----

My first ever published roguelike game is apparently pretty easy.

#### Ray Casting

The algorithm most people first think of when considering how to do realistic lighting is "ray casting". That is, you imagine a circle around the player that is the limit of their light source. For each cell along the edge of the circle, imagine a ray of light emanating from the player towards the center of that cell. Work out what the first object, if any, the ray encounters along its way. All the cells that the ray passes through until the first obstacle are "visible"; the ray casting terminates at that first obstacle.

This algorithm can work, but it has a number of drawbacks. The major drawback is that if the circle is large then the number of rays that must be cast is large. Lots of rays means that the region close to the player is "visited" over and over again, which seems like it's bad for performance. It would be nice if every cell was processed only once, or, almost as good, that every cell is processed only a small number of times.

#### Shadow Casting

The algorithm I've actually implemented here is called "shadow casting"; the basic idea of the algorithm is that rather than tracking the individual rays of light, instead we assume that everything in the circle is lit and then figure out which cells are necessarily in shadow. I'm going to start by describing the algorithm **geometrically**, and then we'll see how the code implementation matches or deviates from the geometrical description.

So let's precisely define some terms here. The world consists of a two-dimensional Euclidean plane of points. Points are represented by pairs of real numbers of the form (x,y). We use the standard geometrical convention that x increases as we move "east" and y increases as we move "north". (\*) The point (0,0) is called the "origin".

The world contains objects that inhabit this plane. Every object is centered on a "lattice point" (x,y) where x and y are both integers, and entirely fills the square bounded by (x-0.5, y - 0.5) in the bottom left corner and (x+0.5, y+0.5) in the top right corner. That region is called the "cell" associated with the lattice point.

The "field of vision" (FoV) problem is to determine which cells are "in line of sight" from a particular point (or, in some algorithms, from *any* point in a given cell) when given a collection of objects that can block sight and a maximum distance.

Without loss of generality, we're going to solve the FoV problem assuming that the "particular point" in question is the origin. As we'll see later, we can do a simple coordinate transformation to solve the problem at other points.

#### 

#### Direction Vectors

The primary tool we're going to use to solve this problem is the "direction vector". A "vector" is like a line segment that emanates from the origin. A vector has both a *magnitude* and a *direction*, but **for our purposes we will only be concerned with the direction**. The direction of a (non-zero-length) vector can be described in a number of ways:

  - Name a point other than the origin which the vector, when extended in a straight line from the origin, would pass through; that determines the direction of the vector.
  - Name a point (x,y) as above; divide the y coordinate of that point by the x coordinate to obtain the "slope".  (\*\*)
  - Draw a unit circle centered on the origin. Extend the vector, if necessary, to pass through the circle. Now measure the distance moving counter-clockwise from the point where the circle touches the x axis to the point where the circle touches the vector. That arc length is the angle of the vector measured in radians. Multiply by 180 / pi to get the angle in degrees.

We could use any of these methods. The disadvantage of the "slope" and "angle" methods is that in our application, slopes and angles will often be fractions that cannot be represented exactly in floating point arithmetic; it would be nice to be able to do all the arithmetic exactly. (Another disadvantage is that slopes increase to infinity as the vector angle approaches 90°, though as we'll see, we'll actually never be working with vectors whose slopes are larger than one or smaller than zero.) In our application it turns out that all the vectors we deal with will pass through a lattice point. We'll therefore use the "name a point" mechanism for characterizing a direction vector.

Here we have a picture illustrating the situation so far. The transparent grey box represents the cell centered on the origin. The filled-in grey box represents an object filling cell (2, 1). The red line represents a direction vector emanating from the origin and passing through (5, 1). (Click on any graph for a larger image.)

[![ShadowCast1](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/2185.ShadowCast1_thumb.png "ShadowCast1")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7801.ShadowCast1_2.png)

#### The Basic Idea of the Algorithm

The basic idea of the algorithm goes like this:

We've already said that without loss of generality, we're going to solve the FoV problem assuming that the cell is the origin. Furthermore, we're going to solve the problem only on **octant zero** of the plane. If you imagine direction vectors at 0°, 45°, 90°, 135°, 180°, 225°, 270° and 315° degrees, you see that those eight vectors divide the plane into eight "octants": (\*\*\*)

[![ShadowCast2](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/4744.ShadowCast2_thumb.png "ShadowCast2")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/5722.ShadowCast2_2.png)

If we can solve the problem in octant zero then we can solve the problem in every other octant by simply reflecting the desired octant into octant zero. For example, to compute FoV in octant seven we could "reflect" all the points in octant seven through the x axis, and hey, now we're in octant zero again.

So, how are we going to solve the problem in octant zero? We will divide the cells whose centers fall into, or on the edges of this octant into **columns: (\*\*\*\*)** Here we see columns zero through six; three of the columns are occupied by opaque cells. The question is, of these cells which are within six units of the origin such that an observer at the origin would have the ability to see the cell?

[![ShadowCast3](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/6724.ShadowCast3_thumb.png "ShadowCast3")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7701.ShadowCast3_2.png)

We're going to go column by column, **left to right**. Within each column we are going to scan from **top to bottom**. We start with a pair of vectors. **The pair of vectors represents a region of the world that is *not* in shadow**.

We start off with the vectors (1,0) and (1,1) because the whole of column zero is in the field of view. Remember, we are interested in the vectors only for their direction, not their magnitude, so I'm going to extend the vectors along their directions indefinitely. When processing column zero, this is the situation:

[![ShadowCast4](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/4645.ShadowCast4_thumb.png "ShadowCast4")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/0451.ShadowCast4_2.png)

The "upper" vector is in blue and the "lower" vector is in green. The cells that fall between these vectors are the ones *thus far* believed to be in the field of view of the origin for this octant. We make a note that cell (0,0) in column zero is visible from column zero.

We then scan column one from **top to bottom**. We do not find anything that would block the view, so the vector state is unchanged after scanning column one, and every cell in column one is visible from the origin. Same for column two. However, when we come to column three we immediately find at the top of column three an opaque cell, but below it there is a transparent cell. We therefore lower the upper vector to account for the fact that the cell at (3,3) is possibly blocking the view of something in a later column.

We do not find anything else opaque in column three, so after processing column three the state of the algorithm now looks like this: (known-to-be-visible cells are marked with a sunburst.)

[![ShadowCast5](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/8461.ShadowCast5_thumb.png "ShadowCast5")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7382.ShadowCast5_2.png)

All of column three was in view, including the opaque cell. But now the upper vector has been lowered. When we start scanning column four, we start from the top, but the top cell in column four is now outside of the region enclosed by the vectors. It is not visible. We start scanning column four from cell (4, 3) downward. We discover that there is an opaque cell at the bottom of column four, so this time we raise the lower vector. At the end of processing column four the state of the algorithm is:

 

[![ShadowCast6](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/8468.ShadowCast6_thumb.png "ShadowCast6")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/4274.ShadowCast6_2.png)

Notice something interesting: the viewable angle is getting smaller and smaller, so even though the columns are getting taller, the actual number of cells we're scanning per column is not growing. This means that it is likely that this algorithm has better performance the more obstacles there are\! That's a nice property to have.

Now we come to the first really interesting part of the algorithm. **Is cell (5,4) visible?** From a strict physics perspective, clearly no portion of cell (5, 4) is visible from an observer exactly at the origin. Any possible line-of-sight vector from the origin either goes through the opaque cell at (3,3) or the opaque cell at (5,3). However, we are scanning each column from the top down; **we haven't processed cell (5, 3) yet**. Another interesting question is: suppose cell (5,3) were transparent; then would cell (5,4) be visible? Its lower right corner would have line of site to the origin, but its center would not. Does that matter? Fortunately, we are saved from having to answer this question because the cell (5, 4) is out of range; we can only see for six units and (5,4) is farther away than that from the origin. However, in general we will need to consider this matter more carefully.

By similar logic, we need to decide whether cell (5,0) is visible or not. Clearly from a "physics" perspective again it is not; it is entirely blocked by cell (4, 0). There might however be implementation or gameplay reasons why we'd want to fudge things a little and allow (5,0) to be visible. We'll come back to these points and consider them in detail when we dive into the exact implementation. For now, let's suppose for the sake of presenting the *idea* of the algorithm than somehow a miracle happens and we consider cell (5, 3) to be the uppermost visible cell of column five and (5,1) to be the lowermost visible cells of column five. As we did when processing column three, we discover that there is a visible opaque cell above a visible transparent cell, and so we lower the upper vector:

[![ShadowCast7](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/1727.ShadowCast7_thumb.png "ShadowCast7")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/8171.ShadowCast7_2.png)

And now there are no cells left that are both less than six units from the origin and have line-of-sight; we're done. The field of view has been determined.

Let's take a briefer look at another scenario. Suppose we have already processed a bunch of columns:

[![ShadowCast8](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7181.ShadowCast8_thumb.png "ShadowCast8")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/4454.ShadowCast8_2.png)

Everything in column four is visible. But what are the vectors to compute the visible cells of columns five and six? We need two sets of vectors now\!

 

[![ShadowCast9](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/3465.ShadowCast9_thumb.png "ShadowCast9")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/1738.ShadowCast9_2.png)

When computing the FoV of columns five and six we'll consider both pairs of vectors as possibly containing viewable area. Naturally, if there were larger columns with many small gaps in them then we could end up generating even more vector pairs.

That's the basic idea of the algorithm; **next time** we'll try to actually implement it in C\# and see what difficulties we run into.

-----

(\*) The fact that many computer display systems do not follow this convention is one of the things that makes it unnecessarily difficult to reason about code that implements these lighting systems. Some implementations assume that when given a rectangle with corners (0,0) and (1,1) the origin is the top left corner, not the bottom left corner as would be conventional geometrically. I think the best thing to do is to follow the geometrical convention, and do a transformation to the display coordinate system in code specifically tasked with making that transformation.

(\*\*) Some implementations of this algorithm that you find on the internet define the "slope" as the *negative of the "run"* divided by the *negative of the "rise"*. Slope is more conventionally defined as the "rise" divided by the "run", as I do here.

(\*\*\*) Some implementations of this algorithm that you find on the internet define octant zero as what I here define as octant two. I think it is more consistent with general practice to number the octants counter-clockwise starting from the x-axis, just as angles are conventionally measured counter-clockwise from the x axis.

(\*\*\*\*) Some implementations call these collections of cells "lines", which is a bit confusing; they are not geometric lines, they are columns of cells.

