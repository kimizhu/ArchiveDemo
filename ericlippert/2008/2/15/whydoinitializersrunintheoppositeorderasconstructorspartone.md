# Why Do Initializers Run In The Opposite Order As Constructors? Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/15/2008 6:01:00 PM

-----

Pop quiz\! What do you expect the output of this program to be?

 

 

using System; class Foo  
{  
    public Foo(string s)  
    {  
        Console.WriteLine("Foo constructor: {0}", s);  
    }  
    public void Bar() { }  
}  
class Base  
{  
    readonly Foo baseFoo = new Foo("Base initializer");  
    public Base()  
    {  
        Console.WriteLine("Base constructor");  
    }  
}  
class Derived : Base  
{  
    readonly Foo derivedFoo = new Foo("Derived initializer");  
    public Derived()  
    {  
        Console.WriteLine("Derived constructor");  
    }  
}  
static class Program  
{  
    static void Main()  
    {  
        new Derived();  
    }  
}

I got a question from a user recently noting that the order was not as he expected. One naively expects that the order will go "base initializers, base constructor body, derived initializers, derived constructor body". In fact the order actually is that first all the initializers run in order *from derived to base*, and then all the constructor bodies run in order *from base to derived*. The latter bit makes perfect sense; the more derived constructors may rely upon state initialized by the less derived constructors, so the constructors should run in order from base to derived. But most people assume that the call sequence of the code above is equivalent to this pseudocode:  

// Expected  
BaseConstructor()  
{  
    ObjectConstructor();  
    baseFoo = new Foo("Base initializer");  
    Console.WriteLine("Base constructor");  
}  
DerivedConstructor()  
{  
    BaseConstructor();  
    derivedFoo = new Foo("Derived initializer");  
    Console.WriteLine("Derived constructor");  
}

When in point of fact it is equivalent to this:

 // Actual  
BaseConstructor()  
{  
    baseFoo = new Foo("Base initializer");  
    ObjectConstructor();  
    Console.WriteLine("Base constructor");  
}  
DerivedConstructor()  
{  
     derivedFoo = new Foo("Derived initializer");  
    BaseConstructor();  
    Console.WriteLine("Derived constructor");  
} 

That explains the *mechanism* whereby the initializers run in order from derived to base and the constructor bodies run in the opposite order, but why did we choose to implement that mechanism instead of the more intuitively obvious former way? Puzzle that one over for a bit, and then read on for a hint. ... ... ...

The "readonly" modifiers in there were no accident. The code gives the appearance that any call to derivedFoo.Bar() and baseFoo.Bar() should never fail with a null dereference exception because both are readonly fields initialized to non-null values.

1.  Is that appearance accurate, or misleading?
2.  Now suppose initializers ran in the "expected" order and answer question (1) again. 

I'll post the answers and analysis next week. Have a fabulous weekend\!

