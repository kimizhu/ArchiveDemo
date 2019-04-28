# High-Dimensional Spaces Are Counterintuitive, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/11/2005 3:24:00 PM

-----

A friend of mine over in Microsoft Research pointed out to me the other day that high-dimensional spaces are really counterintuitive.  He'd just attended a lecture by the research guys who wrote [this excellent paper](http://research.microsoft.com/~jplatt/bitVectors.pdf) and we were geeking out at a party about it.  I found this paper quite eye-opening and I thought I might talk a bit about some of the stuff that's in here at a slightly less highfalutin level -- the paper assumes a pretty deep understanding of high-dimensional spaces from the get-go.  
   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* It's hard to have a geometrical picture in your head of a more than three-dimensional space.  I usually have to use one of two analogies. The first analogy I like goes like this: think of a line -- one dimensional.  Imagine that you have a slider control that determines your position on that line, from, say, -1 to 1, left-to-right.  That's pretty visually clear.  
   
Add another slider that determines your up-and-down position, and you've got a square.  Each point on the square has a unique set of slider positions.  
   
Add another slider that determines your out-and-in position, and you've got a cube.  Again, these are easy to picture.  Every point in the cube has a unique combination of slider positions that gets you to that point.  
   
Now think of a cube with a slider control below it that lets you slide from intense red on one end through dark red and to black on the other.  Now you've got four axes you can move around -- height, width, depth and redness.  The top right front corner of the bright red cube is a certain "colour distance" from the corner of the top right front black cube.  That this is not a spatial dimension isn't particularly relevant; we're just picturing a dimension as redness for convenience.  
   
Every time you want to add another dimension, add another slider -- just make sure that whatever is sliding is completely independent of every other dimension.  Once you've added green and blue sliders then you've got a six-dimensional hypercube. The "distance" between any two 6-d points is a function of how much you have to move how many sliders to get from one to the other. That analogy gets across one of the key ideas of multi-dimensional spaces: that each dimension is simply another independent degree of freedom through which you can move.  But this is a quite mathematical and not very geometric way of thinking about dimensionality, and I want to think about the geometry of these objects.  Let's abandon this analogy.  
   
The second analogy is a little bit more geometrical.  Think of a line, say two units long.  Now associate every point on that line with another line, also two units long "crossing" it at the new line's center.  Clearly that's a filled-in square -- after all, every point along one side of a square has a straight line coming out from it perpendicularly.  In our slider analogy, one slider determines the point along the "main line", and the second determines how far to go along its associated line. Think of another line, but this time, associate every point on it with a square.  That's a solid cube.  Now think of yet another line.  Associate every point on it with a cube, and you've got a 4-cube.  At this point it gets hard to visualize, but just as a cube is an infinite number of equally-sized squares stuck together along a line, so is a 4-cube an infinite number of 3-cubes stuck together along a line.  Similarly, a 5-cube is a line of 4-cubes, and so on.  
   
Where things get weird is when you start to think about hyperspheres instead of hypercubes.  Hyperspheres have some surprising properties that do not match our intuition, given that we only have experience with two and three dimensional spheres. (2-spheres are of course normally called "circles".)  
   
The definition of a hypersphere is pretty simple -- like a 2-sphere or 3-sphere, a hypersphere is the collection of points that are all the same distance from a given center point. (But distance works strangely in higher dimensions, as we'll see in future episodes\!)  
   
Its hard to picture a hypercube; it's even hard to picture a hypersphere. The equivalent analogy for n-spheres requires us to think about size.  Again, imagine a line two units long.  Associate with each point on the line another line crossing at the middle.  But this time, the associated lines are of different lengths.  The lines associated with the end points are tiny, and the lines associated with the middle are longer.  This describes a circular disk -- for each point along the diameter of a circle you can draw a perpendicular line through the point extending to the boundaries of the disk on each side. Now do the same thing again.  Take a line, and associate each point on the line with a circle.  If the circles are all the same size, you have a cylinder.  But if they vary from small at the ends to big in the middle, you've got a sphere.  Successive cross-sections of a sphere are all circles, but they start small and get big and then get small again.  
   
Now do the same thing again.  Take a line and associate each point on the line with a sphere, small at the ends and big in the middle, and you've got a 4-sphere.  Successive "cross sections" of a 4-sphere are 3-spheres of varying size.  Keep going to 5-, 6-, etc, spheres.  
   
A circle of diameter 2 fits into a square of edge length 2, and a sphere of diameter 2 fits into a cube of edge length 2.  Clearly an n-sphere of diameter two fits exactly into an n-cube of edge length two -- the n-sphere "kisses" the center of each face of the n-cube.  You can't make the n-cube smaller without the n-sphere poking out of it somewhere.    
   
But things start getting weird when you consider the volume of an n-sphere. Tomorrow we'll compare the volume of an n-sphere to the volume of an n-cube, and discover some surprising and counterintuitive things about where that volume is.

