# Shadowcasting in C\#, Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/29/2011 7:05:00 AM

-----

OK, let's finish up this year and this series. We have an algorithm that can compute what cells in the zero octant are in view to a viewer at the origin when given a function that determines whether a given cell is opaque or transparent. It marks the visible points by calling an action with the visible cells. We would like that to work in any octant, and for the viewer at any point, not just the origin.

We can solve the "viewer at any point" problem by imposing a **coordinate transformation** on the "what is opaque?" function and the "cell is visible" action. Suppose the algorithm wishes to know whether the cell (3, 1) is opaque. And suppose the viewer is not at the origin, but rather at (5, 6). The algorithm is actually asking whether the cell at (3 + 5, 6 + 1) is opaque. Similarly, if that cell is determined to be visible then the transformed cell is the one that is visible from (5, 6). We can easily transform one delegate into another:

private static Func\<int, int, T\> TranslateOrigin\<T\>(Func\<int, int, T\> f, int x, int y)  
{  
    return (a, b) =\> f(a + x, b + y);  
}

private static Action\<int, int\> TranslateOrigin(Action\<int, int\> f, int x, int y)  
{  
    return (a, b) =\> f(a + x, b + y);  
}

Similarly we can perform an octant transformation; if you want to map a point in octant one into a point in octant zero, just swap its (x,y) coordinates\! Every octant has a simple transformation that reflects it into octant zero:

private static Func\<int, int, T\> TranslateOctant\<T\>(Func\<int, int, T\> f, int octant)  
{  
    switch (octant)  
    {  
        default: return f;  
        case 1: return (x, y) =\> f(y, x);  
        case 2: return (x, y) =\> f(-y, x);  
        case 3: return (x, y) =\> f(-x, y);  
        case 4: return (x, y) =\> f(-x, -y);  
        case 5: return (x, y) =\> f(-y, -x);  
        case 6: return (x, y) =\> f(y, -x);  
        case 7: return (x, y) =\> f(x, -y);  
    }  
}

(And similarly for actions.)

Now that we have these simple transformation functions we can finally implement the code that calls our octant-zero-field-of-view algorithm:

public static void ComputeFieldOfViewWithShadowCasting(  
    int x, int y, int radius,  
    Func\<int, int, bool\> isOpaque,  
    Action\<int, int\> setFoV)  
{  
    Func\<int, int, bool\> opaque = TranslateOrigin(isOpaque, x, y);  
    Action\<int, int\> fov = TranslateOrigin(setFoV, x, y);

    for (int octant = 0; octant \< 8; ++octant)  
    {  
        ComputeFieldOfViewInOctantZero(  
            TranslateOctant(opaque, octant),  
            TranslateOctant(fov, octant),  
            radius);  
    }  
}  

Pretty slick, eh?

One minor downside of this algorithm is that it computes the visibility of the points along the axes and major diagonals twice; however, the number of such cells grows worst case linearly with radius. The algorithm as a whole is worst case quadradic in the radius, so the extra linear cost is probably irrelevant.

I've posted [the project that builds the Silverlight control from the first episode here](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/1777.SilverlightShadowCasting.zip), if you want to get a complete working example.

Happy New Year everyone and we'll see you in 2012 for more Fabulous Adventures in Coding\!

