# Debunking another myth about value types

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/11/2010 8:53:00 AM

-----

Here's another myth about value types that I sometimes hear:

"Obviously, using the new operator on a reference type allocates memory on the heap. But a value type is called a value type because it stores its own value, not a reference to its value. Therefore, using the new operator on a value type allocates no additional memory. Rather, the memory already allocated for the value is used."

That seems plausible, right? Suppose you have an assignment to, say, a field s of type S:

 

s = new S(123, 456);

If S is a reference type then this allocates new memory out of the long-term garbage collected pool, a.k.a. "the heap", and makes s refer to that storage. But if S is a value type then there is no need to allocate new storage because we already have the storage. The variable s already exists and we're going to call the constructor on it, right?

Wrong. That is not what the C\# spec says and not what we do. (Commenter Wesner Moise points out that yes, that is sometimes what we do. More on that in a minute.)

It is instructive to ask "what if the myth were true?" Suppose it were the case that the statement above meant "determine the memory location to which the constructed type is being assigned, and pass a reference to that memory location as the 'this' reference in the constructor". Consider the following class defined in a single-threaded program (for the remainder of this article I am considering only single-threaded scenarios; the guarantees in multi-threaded scenarios are much weaker.)

 

using System;  
struct S  
{  
    private int x;  
    private int y;  
    public int X { get { return x; } }  
    public int Y { get { return y; } }  
    public S(int x, int y, Action callback)  
    {  
        if (x \> y)  
            throw new Exception();  
        callback();  
        this.x = x;  
        callback();  
        this.y = y;  
        callback();  
    }  
}

We have an immutable struct which throws an exception if x \> y. Therefore it should be impossible to ever get an instance of S where x \> y, right? That's the point of this invariant. But watch:

 

static class P  
{  
    static void Main()  
    {  
        S s = default(S);  
        Action callback = ()=\>{Console.WriteLine("{0}, {1}", s.X, s.Y);};  
        s = new S(1, 2, callback);  
        s = new S(3, 4, callback);  
    }  
}

Again, remember that we are supposing the myth I stated above to be the truth. What happens?

\* First we make a storage location for s. (Because s is an outer variable used in a lambda, this storage is on the heap. But the location of the storage for s is irrelevant to today's myth, so let's not consider it further.)  
\* We assign a default S to s; this does not call any constructor. Rather it simply assigns zero to both x and y.  
\* We make the action.  
\* We (mythically) obtain a reference to s and use it for the 'this' to the constructor call. The constructor calls the callback three times.  
\* The first time, s is still (0, 0).  
\* The second time, x has been mutated, so s is (1, 0), violating our precondition that X is not observed to be greater than Y.  
\* The third time s is (1, 2).  
\* Now we do it again, and again, the callback observes (1, 2), (3, 2) and (3, 4), violating the condition that X must not be observed to be greater than Y.

This is horrid. We have a perfectly sensible precondition that looks like it should never be violated because we have an immutable value type that checks its state in the constructor. And yet, in our mythical world, it is violated.

Here's another way to demonstrate that this is mythical. Add another constructor to S:

 

    public S(int x, int y, bool panic)  
    {  
        if (x \> y)  
            throw new Exception();  
        this.x = x;  
        if (panic)  
            throw new Exception();  
        this.y = y;  
    }  
}

We have

static class P  
{  
    static void Main()  
    {  
        S s = default(S);  
        try  
        {  
            s = new S(1, 2, false);  
            s = new S(3, 4, true);  
        }  
        catch(Exception ex)  
        {  
            Console.WriteLine("{0}, {1}", s.X, s.Y);};  
        }  
    }  
}

Again, remember that we are supposing the myth I stated above to be the truth. What happens? If the storage of s is mutated by the first constructor and then partially mutated by the second constructor, then again, the catch block observes the object in an inconsistent state. Assuming the myth to be true. **Which it is not.** The mythical part is right here:

Therefore, using the new operator on a value type allocates no additional memory. Rather, the memory already allocated for the value is used.

That's not true, and as we've just seen, if it were true then it would be possible to write some really bad code. The fact is that both statements are false. The C\# specification is clear on this point:

"If T is a struct type, an instance of T is created by allocating a temporary local variable"

That is, the statement

s = new S(123, 456);

actually means:

\* Determine the location referred to by s.  
\* Allocate a temporary variable t of type S, initialized to its default value.  
\* Run the constructor, passing a reference to t for "this".  
\* Make a by-value copy of t to s.

This is as it should be. The operations happen in a predictable order: first the "new" runs, and then the "assignment" runs. In the mythical explanation, there is no assignment; it vanishes. And now the variable s is never observed to be in an inconsistent state. The only code that can observe x being greater than y is code in the constructor. Construction followed by assignment becomes "atomic"(\*).

In the real world if you run the first version of the code above you see that s does not mutate until the constructor is done. You get (0,0) three times and then (1,2) three times. Similarly, in the second version s is observed to still be (1,2); only the temporary was mutated when the exception happened.

Now, what about Wesner's point? Yes, in fact if it is a stack-allocated local variable (and not a field in a closure) that is declared at the same level of "try" nesting as the constructor call then we do not go through this rigamarole of making a new temporary, initializing the temporary, and copying it to the local. In that specific (and common) case we *can* optimize away the creation of the temporary and the copy because it is impossible for a C\# program to observe the difference\! But *conceptually* you should think of the creation as a creation-then-copy rather than a creation-in-place; that it sometimes can be in-place is an implementation detail that you should not rely upon.

\----------------------------

(\*) Again, I am referring to single-threaded scenarios here. If the variable s can be observed on different threads then it can be observed to be in an inconsistent state because copying any struct larger than an int is not guaranteed to be a threadsafe atomic operation.

