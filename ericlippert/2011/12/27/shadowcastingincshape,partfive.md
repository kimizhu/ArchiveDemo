# Shadowcasting in C\#, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/27/2011 10:05:00 AM

-----

I hope you all had a pleasant Christmas and Boxing Day; we chose to not travel to see family this year and had a delightful time visiting friends. We'll finish up 2011 here with a bit more on shadowcasting, and then pick up with more C\# language design facts and opinions in January.

-----

OK, so we've found the top and bottom cells in a particular column portion, bounded by a top and bottom vector. Now we have two tasks. First, all cells in that portion that are in the radius need to be marked as visible. Second, we must figure out what portions of the next column are visible through this portion, and enqueue that work.

The first bit is easily done. Thanks to the great comments to my earlier entry I've learned that of course it is better to consider the question "is the bottom left corner of the cell within the visible radius?" That makes for a nicer looking circle. I've made a helper method "IsInRadius" that does that. (Note that of course this introduces yet another kind of artifact; suppose that of the four corners of a cell, only the bottom right corner of a cell is below the top vector, but only the bottom left corner is within the radius. There might be no point in the cell that is both within the radius and not blocked. We'll ignore this detail; the radius is inherently approximate.)

The second bit is harder. What we'll do is keep track as we move from top to bottom of this portion whether we have seen any transitions from transparent to opaque or opaque to transparent. If we find a transition from transparent to opaque then we have found a boundary for the next column portion; we'll enqueue that new portion. If we find a transition from opaque to transparent then we'll enqueue new work for the new region when either we find an opaque cell or the bottom of the portion.

bool? wasLastCellOpaque = null;  
for (int y = topY; y \>= bottomY; --y)  
{  
    bool inRadius = IsInRadius(x, y, radius);  
    if (inRadius)  
    {  
        // The current cell is in the field of view.  
        setFieldOfView(x, y);  
    }

A cell that was too far away to be seen is effectively an opaque cell; nothing "above" it is going to be visible in the next column, so we might as well treat it as an opaque cell and not scan the cells that are also too far away in the next column.

    bool currentIsOpaque = \!inRadius || isOpaque(x, y);  
    if (wasLastCellOpaque \!= null)  
    {  
        if (currentIsOpaque)  
        {

We've found a boundary from transparent to opaque. Enqueue more work for later.

            if (\!wasLastCellOpaque.Value)  
            {

The new bottom vector touches the upper left corner of opaque cell that is below the transparent cell.

                queue.Enqueue(new ColumnPortion(  
                    x + 1,  
                    new DirectionVector(x \* 2 - 1, y \* 2 + 1),  
                    topVector));  
            }  
        }  
        else if (wasLastCellOpaque.Value)  
        {

We've found a boundary from opaque to transparent. Adjust the top vector so that when we find the next boundary or do the bottom cell, we have the right top vector. The new top vector touches the lower right corner of the opaque cell that is above the transparent cell, which is the upper right corner of the current transparent cell.

I normally don't modify a formal parameter like this, but it seems reasonably unconfusing in this context.

            topVector = new DirectionVector(x \* 2 + 1, y \* 2 + 1);  
        }  
    }  
    wasLastCellOpaque = currentIsOpaque;  
}

And finally, we enqueue work for the lowest opaque-to-transparent transition, if there is one.

if (wasLastCellOpaque \!= null && \!wasLastCellOpaque.Value)  
    queue.Enqueue(new ColumnPortion(x + 1, bottomVector, topVector));  

And there you go; that's shadowcasting from the origin in the first octant. Next time we'll deal with the other seven octants, and deal with points other than the origin.

