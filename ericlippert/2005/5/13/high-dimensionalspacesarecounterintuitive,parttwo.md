# High-Dimensional Spaces Are Counterintuitive, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/13/2005 2:27:00 PM

-----

The volume of an n-cube of edge length s is easy to work out. A 2-cube has s<sup>2</sup> units of area. A 3-cube has s<sup>3</sup> units of volume. A 4-cube has s<sup>4</sup> units of 4-volume, and so on -- an n-cube has s<sup>n</sup> units of n-volume. If the n-cube has edge of s\>1, say s=2, then clearly the n-volume dramatically increases as the dimensionality increases -- each dimension adds a lot more "room" to the n-cube.

A 2-sphere (ie, circle) is pretty close in area to the smallest 2-cube (ie, square) that encloses it -- sure, you lose some area at the four corners, but not a whole lot. Though the circle is far from the square at the four corner, it is very close to the square at the four sides. A circle has about 75% the area of its enclosing square. But a 3-sphere inside a the smallest 3-cube that encloses it is far from eight corners and close to only six sides. A 3-sphere is about half the volume of the 3-cube. As you go up in dimensions, you get more and more corners that are far from the n-sphere -- there are 2<sup>n</sup> corners and only 2n sides so the comparative volume of the sphere goes down.

In fact, you don't even need to compare n-spheres to n-cubes -- after you reach 5 dimensions, the n-volume of an n-sphere starts going down, not up, as dimensionality increases. With some pretty easy calculus you can show that the n-volume of an n-sphere of radius r is:

V<sub>1</sub> = 2 r  
V<sub>2</sub> = π r<sup>2</sup>  
V<sub>n</sub> = V<sub>n-2</sub> 2 π r<sup>2</sup> / n

For any fixed radius this rapidly approaches zero as n gets big. Pick a big n, say 100. The volume of a 100-sphere is going to be (π r<sup>2</sup> /50) x (π r<sup>2</sup> /49) x (π r<sup>2</sup>/48) ... (π r<sup>2</sup>/1). Suppose r is 1 -- then all of those terms except for the last few are going to be quite a bit smaller than 1. Every time you add more dimensions, the n-volume of the unit n-sphere gets smaller and smaller even as the n-volume of the smallest n-cube that encloses the unit n-sphere gets exponentially larger and larger\!

Here's another weird fact about the volume of hypersolids. Consider two squares, one inside the other. How big does the small square have to be in order to have, say, 1% the area of the larger square? That's pretty easy. If the inner square has 10% the edge length of the outer square, then it has 1% of the area of the outer square.

What about nested 3-cubes? An inner 3-cube with edges 10% the length of the edge of the outer 3-cube would have 0.1% the volume, too small. Rather, it needs to havean edge about 21% of the edge of the outer 3-cube, because .21 x .21 x .21 = about 0.01.

What about a nested n-cube? In order to have 1% the n-volume of the outer n-cube, the inner n-cube needs to have an edge of (0.01)<sup>1/n</sup> of the outer n-cube side. For n=100, that's 0.955. Think about that for a moment. You've got two 100-cubes, one has edges 2 units long, the other has edges 1.91 units long. The larger n-cube contains ONE HUNDRED TIMES more volume.

Try to visualize the smaller n-cube being entirely inside the larger n-cube, the two n-cubes having the same central point. Now wrap your mind around the fact that the smaller n-cube is 1% the volume of the larger. The conclusion is unavoidable: **in high dimensions the vast majority of the volume of a solid is concentrated in a thin shell near its surface\!** Remember, there's 2<sup>100</sup> corners in a 100-cube, and that makes for a lot of space to put stuff.

It's counterintuitive because the very idea of "near" is counterintuitive in higher dimensions. Every time you add another dimension, there's more room for points to be farther apart.

The distance between opposite corners of a square of edge 2 is 2√2. The distance between opposite corners of a 3-cube is 2√3, quite a bit bigger. The distance between opposite corners of a 100-cube is 2√100 = 20 units\! There are a whole lot of dimensions to move through, and that adds distance.

We could make the same argument for an n-sphere and show that the vast majority of its (comparatively tiny) volume is also in a thin shell near the surface; I'm sure you can see how the argument would go, so I won't bother repeating myself.

Because distance is so much more "expensive" in higher dimensions, this helps explain why n-spheres have so much less volume than n-cubes. Consider a 100-cube of edge 2 centered on the origin enclosing a 100-sphere of diameter 2, also centered on the origin. The point (1,0,0,0,0,0...,0) is on both the 100-cube and the sphere, and is 1 unit from the origin. The point (1,1,1,...,1) is on the 100-cube and is ten units away from the origin. But a 100-sphere by definition is the set of points equidistance from the origin, and distance is expensive in high dimensions. The nearest point on the 100-sphere to that corner is (0.1, 0.1, 0.1, ..., 0.1), 9 units away from the corner of the 100-cube. Now its clear just how tiny the 100-sphere is compared to the 100-cube.

OK, so far we've been considering n-cubes that entirely enclose n-spheres, ie, an n-cube of edge length 2 that encloses a unit n-sphere, kissing the sphere at 2n points. But we know that this n-cube has ginormously more volume than the n-sphere it encloses and that most of that volume is near the edges and corners. What if we abandon the constraint that the n-cube contains 100% of the n-sphere's volume. After all, there are only 200 points where the 100-sphere kisses the 100-cube, and that's not very many at all.

Suppose we want an 100-cube that contains 99.9% of the volume of the unit 100-sphere. We can cover virtually all of the volume of the 100-sphere with an 100-cube of edge 0.7 instead of 2. Sure, we're missing (1,0,0,0,...,0), but we're still hitting (0.1,0.1,0.1,...) with huge amounts of room to spare. Most of the volume inside the 100-sphere isn't near the 200 points with coordinates near the axes.

How much do we reduce the volume of the 100-cube by shrinking it from 2 on the edge to 0.7? We go from 2<sup>100</sup> n-units of volume to 0.7<sup>100</sup>, a factor of around 4x10<sup>45</sup> times smaller volume\! And yet we still enclose virtually all the volume of the 100-sphere. The corner of the smaller 100-cube at (0.35, 0.35, 0.35, ...) is now only 2.5 units away from (0.1, 0.1, ...) instead of 9 units away. This is a much better approximation of the unit 100-sphere. It's still hugely enormous compared to the unit 100-sphere in terms of sheer volume, but look at how much volume we save by approximating the 100-sphere as a small 100-cube\!

Feeling dizzy yet? Next time we'll see that these facts

  - n-spheres are tiny compared to n-cubes
  - hypersolids have most of their volume close to their surfaces
  - you can enclose almost all the volume of an n-sphere with a small n-cube

have major repercussions when we throw probability theory into the mix.

