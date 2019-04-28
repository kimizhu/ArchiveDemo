# Shadowcasting in C\#, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/15/2011 8:43:00 AM

-----

I hope the basic idea of the shadow casting algorithm is now clear. Let's start to implement the thing. There are two main concerns to deal with. The easy one is "what should the interface to the computation look like?" The second is "how to implement it?" Let's deal with the easy one first; let's design the API.

What does the caller need to provide?

  - The coordinates of a central point
  - The radius of the field of view
  - Some way for the algorithm to know which cells are opaque

What does the implementation need to do for the caller?

  - Provide some way of telling the caller which cells are visible from the central point.

It's that last one that is a bit tricky. The implementation could return a list of point objects that are in view. Or it could create a two-dimensional array of bools and set the bools to true if the cell is in view or false if it is not. It could mutate a caller-provided collection. And so on. We don't know how the caller works or what it is going to do with that information. We don't even know if it is storing that information as bools or bit flags or a list of points. It is hard to know what the right thing to do is, so we'll punt on it. We'll make the caller decide by making the caller pass in an Action that does the right thing for it\!

public static class ShadowCaster  
{  
    // Takes a circle in the form of a center point and radius, and a function that  
    // can tell whether a given cell is opaque. Calls the setFoV action on  
    // every cell that is both within the radius and visible from the center.  
    public static void ComputeFieldOfViewWithShadowCasting(  
        int x, int y, int radius,  
        Func\<int, int, bool\> isOpaque,  
        Action\<int, int\> setFoV)  
    {  
        // The miracle happens here  
    }

OK, so that's the point of entry for the caller. What about the implementation?

I wanted my implementation to have the following characteristics:

First and foremost, the implementation should be **clear** and **correct**. It should be **performant enough** for small demos, but not necessarily wringing every last drop of performance out of the processor. If the code is clear and correct but not fast enough, targeted performance analysis can find the hot spot later. For debuggability, I'd like it if the code operates more or less in **the same order as in the description of the algorithm** I laid out. Also, the code should be **DRY** -- Don't Repeat Yourself. (\*)

I want the implementation to not be overly concerned with vexing book-keeping details. We laid out the algorithm as one which assumed that the viewpoint was the origin and the field of view was calculated only in the zero octant; our implementation should do the same, rather than trying to keep track of details like where the viewpoint really is.

This algorithm is often implemented recursively but I wanted to avoid that, for two reasons. First, because the typical recursive implementation **recurses at least once per column**; one can imagine a scenario in which a long narrow tunnel hundreds of cells long blows the stack. Second, because the typical recursive implementation explores the octant in a "column depth first" manner. That is, when it must divide the visible region into multiple "portions" each with its own top and bottom direction vector, it explores each portion through to the final column; the priority is to explore each *portion* entirely before starting on the next. But we characterized the algorithm as a straightforward **left-to-right, top-to-bottom** progression of cells that explores each *column* entirely before starting on the next. As I said before, for both clarity and debuggability it would be nice if the implementation matched the description.

The basic idea of my implementation goes like this:

  - For each column, take as an input a set of cells in a column known to be either definitely in the field of view, or possibly just barely out-of-radius.
  - From that set, compute which cells in the *next* column are either definitely in the field of view or possibly just out-of-radius.
  - Repeat until you get to the column that is entirely outside of the field-of-view radius; you can stop there.

That's a good high-level overview, but let's make the action a bit more crisp:

  - Break each column (identified by the x-axis coordinate that defines the center of the column) down into one or more contiguous "portions" each bounded by a top and bottom direction vector.
  - For each portion in the current column, determine the set of portions *in the subsequent column* that are visible.
  - Add each of those subsequent portions to a work queue.
  - Keep on processing portions from the work queue until there are no more.

OK, that's enough of a description to actually write some code to implement these abstractions. We can do that with two little immutable structs.

Recall that we decided to represent direction vectors as a point on the line of the vector, and that we do not care about the magnitude, only the direction. As we'll see, the only direction vectors we need fall on lattice points, so we can use ints as the coordinates.

private struct DirectionVector  
{  
    public int X { get; private set; }  
    public int Y { get; private set; }  
    public DirectionVector(int x, int y)  
        : this()  
    {  
        this.X = x;  
        this.Y = y;  
    }  
}

The portion of the column we are dealing with is characterized by three facts: what is the x-coordinate of the column's center, what is the direction vector bounding the top of the portion, and what is the direction vector bounding the bottom of the portion?

private struct ColumnPortion  
{  
    public int X { get; private set; }  
    public DirectionVector BottomVector { get; private set; }  
    public DirectionVector TopVector { get; private set; }  
    public ColumnPortion(int x, DirectionVector bottom, DirectionVector top)  
        : this()  
    {  
        this.X = x;  
        this.BottomVector = bottom;  
        this.TopVector = top;  
    }  
}

Now that we have these data structures we can make the main loop of the engine. Note that we are now assuming that the center point is the origin and that we are only interested in octant zero. Somehow the entry point is going to have to figure out how to deal with that requirement, but that's a problem that we'll solve later.

private static void ComputeFieldOfViewInOctantZero(  
    Func\<int, int, bool\> isOpaque,  
    Action\<int, int\> setFieldOfView,  
    int radius)  
{  
    var queue = new Queue\<ColumnPortion\>();  
    queue.Enqueue(new ColumnPortion(0, new DirectionVector(1, 0), new DirectionVector(1, 1)));  
    while (queue.Count \!= 0)  
    {  
        var current = queue.Dequeue();  
        if (current.X \>= radius)  
            continue;  
        ComputeFoVForColumnPortion(  
            current.X,  
            current.TopVector,  
            current.BottomVector,  
            isOpaque,  
            setFieldOfView,  
            radius,  
            queue);  
    }  
}

The action of the main loop is straightforward. We make a work queue. We know that all of column 0 is in the field of view and that its top and bottom vectors are the lines emanating from the origin that bound the entire octant. We put that on the work queue. We then sit in a loop taking work off the queue and processing each portion of the column. Doing so may put arbitrarily more work on the queue for the next column. Since the work queue is a queue, we guarantee that we complete one column before we start working on the next; this makes the action of the algorithm similar to that of the description of the algorithm.

The attentive reader will have noticed that we've already made a very interesting choice that actually fails to correctly implement the stated algorithm. If the column portion on the queue is outside of the radius of the field of view then we discard it without processing it. This guarantees that the algorithm will terminate, and also makes sure that we don't do unnecessary work computing a column that is entirely outside of the field of view. That in of itself is fine; the interesting choice is that the comparison is

if (current.X \>= radius)

and not

if (current.X \> radius)

If we are asked for a field of view of radius six **we do not actually make any cells in column six visible** even though exactly one of them might be visible -- namely, the cell at (6, 0). Every other cell in that column is more than six units away from the origin. Why make this choice?

Aesthetics. Suppose there are no obstacles, and we compute the field of view of radius six for all eight octants. The resulting field of view will look like this:

       O        
    OOOOOOO  
   OOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
 OOOOOO@OOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
   OOOOOOOOO  
    OOOOOOO  
       O         

Which looks *bizarre*. The curvature of a circle by definition should appear to be the same everywhere; this makes the circle look extremely pointy at four places. The boundary of a circle should be convex everywhere; if you imagine joining the center points of all the O's along the boundary they make for a convex hull except at eight points where the circle suddenly becomes concave. This is terribly ugly; to eliminate this ugliness we round off to an octagon by omitting the extreme column:

                
    OOOOOOO  
   OOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
  OOOOO@OOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
  OOOOOOOOOOO  
   OOOOOOOOO  
    OOOOOOO  
               

Much nicer. And the "error" is small, both in that it is only four points that are removed, and small in the sense that these are the four points that are the farthest-away points visible from the center; if you're going to eliminate points, those are the most sensible ones to take away.

Today we saw that a small rounding decision can have a big impact on the aesthetics of the algorithm; **next time** we'll dig into the first statement of ComputeFoVForColumnPortion and discover that subtle decisions about managing rounding errors can make a big difference in determining how the output looks to the player.

-----

(\*) Many implementations of this algorithm you find on the internet needlessly repeat all of the code eight times, once for each octant.

