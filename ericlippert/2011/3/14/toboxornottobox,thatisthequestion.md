# To box or not to box, that is the question

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/14/2011 7:20:00 AM

-----

Suppose you have an immutable value type that is also disposable. Perhaps it represents some sort of handle.

 

struct MyHandle : IDisposable  
{  
    public MyHandle(int handle) : this() { this.Handle = handle; }  
    public int Handle { get; private set; }  
    public void Dispose()  
    {  
        Somehow.Close(this.Handle);  
    }  
}

You might think hey, you know, I'll decrease my probability of closing the same handle twice by making the struct mutate itself on disposal\!

 

public void Dispose()  
{  
    if (this.Handle \!= 0)  
      Somehow.Close(this.Handle);  
    this.Handle = 0;  
}

This should already be raising red flags in your mind. We're mutating a value type, which we know is dangerous because value types are copied by value; you're mutating a variable, and different variables can hold copies of the same value. Mutating one variable does not mutate the others, any more than changing one variable that contains 12 changes every variable in the program that also contains 12. But let's go with it for now.

What does this do?

 

var m1 = new MyHandle(123);  
try  
{  
  // do something  
}  
finally  
{  
    m1.Dispose();  
}  
// Sanity check  
Debug.Assert(m1.Handle == 0);

Everything work out there?

Yep, we're good. m1 begins its life with Handle set to 123, and after the dispose it is set to zero.

How about this?

 

var m2 = new MyHandle(123);  
try  
{  
  // do something  
}  
finally  
{  
    ((IDisposable)m2).Dispose();  
}  
// Sanity check  
Debug.Assert(m2.Handle == 0);

Does that do the same thing? Surely casting an object to an interface it implements does nothing untoward, right?

.

.

.

.

.

.

.

.

Wrong. This boxes m2. Boxing makes a copy, and it is the copy which is disposed, and therefore the copy which is mutated. m2.Handle stays set to 123.

So what does this do, and **why**?

 

var m3 = new MyHandle(123);  
using(m3)  
{  
  // Do something  
}  
// Sanity check  
Debug.Assert(m3.Handle == 0);

.

.

.

.

.

.

.

.

.

.

 

Based on the previous example you probably think that this boxes m3, mutates the box, and therefore the assertion fires, right?

Right?

Is that what you thought?

You'd be perfectly justified in thinking that there is a boxing performed in the finally because **that's what the spec says**. The spec says that the "using" statement's expansion when the expression is a non-nullable value type is

 

finally  
{  
  ((IDisposable)resource).Dispose();  
}

However, I'm here today to tell you that **the disposed resource is in fact not boxed** in our implementation of C\#. The compiler has an optimization: if it detects that the Dispose method is exposed directly on the value type then it effectively generates a call to

 

finally  
{  
  resource.Dispose();  
}

without the cast, and therefore without boxing.

Now that you know that, would you like to change your answer? **Does the assertion fire? Why or why not?**

Give it some careful thought.

.

.

.

.

.

.

.

.

.

.

**The assertion still fires, even though there is no boxing**. The relevant line of the spec is not the one that says that there's a boxing cast; that's a red herring. The relevant bit of the spec is:

 

A using statement of the form "using (ResourceType resource = expression) statement" corresponds to one of three possible expansions. \[...\] A using statement of the form "using (expression) statement" has the same three possible expansions, but in this case ResourceType is implicitly the compile-time type of the expression, and the resource variable is inaccessible in, and invisible to, the embedded statement.

That is to say, our program fragment is equivalent to:

 

var m3 = new MyHandle(123);  
using(MyHandle invisible = m3)  
{  
  // Do something  
}  
// Sanity check  
Debug.Assert(m3.Handle == 0);

which is equivalent to

 

var m3 = new MyHandle(123);  
{  
  MyHandle invisible = m3;  
  try  
  {  
    // Do something  
  }  
  finally  
  {  
    invisible.Dispose(); // No boxing, due to optimization  
  }  
}  
// Sanity check  
Debug.Assert(m3.Handle == 0);

**It is the invisible copy which is disposed and mutated, not m3.**

And that's why the compiler can get away with not boxing in the finally. **The thing that it is not boxing is invisible and inaccessible and therefore there is no way to observe that the boxing was skipped**.

Once again the moral of the story is: **mutable value types are [enough pure evil to turn you all into hermit crabs,](https://www.youtube.com/watch?v=F6X9KcrXHwg) and therefore should be avoided.**

