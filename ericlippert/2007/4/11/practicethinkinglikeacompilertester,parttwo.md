# Practice thinking like a compiler tester, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/11/2007 10:00:00 AM

-----

Reader RichM found the same solution to [the puzzle I posed yesterday](http://blogs.msdn.com/ericlippert/archive/2007/04/10/practice-thinking-like-a-compiler-tester.aspx) that I did:

 

public class V : S.T {}  
public class S {  
    public class T{}  
}

The control flow of the emitter from the start to the point of the bug goes like this:

 

Emit(V)  
    Emit (S.T) // V base class  
        Emit(null) // S.T base class  
        Emit(S) // S.T outer class  
            Emit(null) // S base class  
            Emit(null) // S outer class  
            ReallyEmit(S)  
            S.emitted = true  
            Emit(S.T) // S's first inner class  
                Emit(S) // S.T's outer class.  
                    // Take the early out because S.emitted is true  
                ReallyEmit(S.T)  
                S.T.emitted = true  
                // T has no inner classes, so return  
            // S has no more inner classes, so return  
        ReallyEmit(S.T) // oops, we already did this. Bug\!

Of course this was not what the emitting code actually looked like. What it actually looked like was something like:

 

void Emit(Class c)  
{  
    if (c == null || c.Emitted) return; // don’t emit twice\!  
    Emit(c.BaseClass);  
    Emit(c.OuterClass);  
    // If somehow Emit was called on this class before our outer class, then  
    // we just caused ourselves to be emitted when we emitted the outer class.  
    // (The outer class emits its inner classes, ie, the current  
    // class, before it returns.)  
    // If we are already emitted then we have no more work to do.  
    if (c.Emitted) {  
        Debug.Assert(c.OuterClass \!= null && c.OuterClass.Emitted);  
        return;  
    }  
    ReallyEmit(c);   
    c.Emitted = true;  
    foreach(Class i in c.InnerClasses)  
        Emit(i);  
}

This algorithm is correct. However, **the comment and the assertion are not correct**. (This is why I was looking at this code in the first place, because the assertion was firing. I fixed it by updating the comment and removing the assertion.)

Can you find a legal set of C\# classes which are correctly emitted but cause the assertion to fire?

