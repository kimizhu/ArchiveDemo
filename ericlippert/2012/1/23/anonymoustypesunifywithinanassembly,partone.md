# Anonymous types unify within an assembly, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/23/2012 7:55:00 AM

-----

Back [in my last post of 2010 I said that I would do an example of anonymous types unifying within an assembly](http://blogs.msdn.com/b/ericlippert/archive/2010/12/20/why-are-anonymous-types-generic.aspx) "in the new year". I meant 2011, but here we are "in the new year" again, so, no time like the present.

The C\# specification guarantees you that when you use "the same" anonymous type in two places in one assembly (\*) the types unify into one type. In order to be "the same", the two anonymous types have to have the exact same member names and the exact same member types, in the exact same order. (\*\*)

It is easy to see how this works in a method body; you could simply say:

var anon1 = new { X = 1 };  
var anon2 = new { X = 2 };  
anon2 = anon1;  
Console.WriteLine(anon2.X); // 1

And it is easy to see how you could verify this fact with reflection across method bodies within the same assembly:

class C  
{  
    public static object Anon1() { return new { X = 1 }; }  
}  
class D  
{  
    static void M()  
    {  
        // True:  
        Console.WriteLine(C.Anon1().GetType() == (new { X = 2 }).GetType());  
    }  
}

But... so what? Using reflection in this way seems supremely uninteresting. What would be more interesting is to merge the two examples together:

class C  
{  
    public static object Anon1() { return new { X = 1 }; }  
}  
class D  
{  
    static T CastByExample\<T\>(T example, object obj)  
    {  
        return (T)obj;  
    }  
    static void M()  
    {  
        var anon1 = CastByExample(new { X = 0 }, C.Anon1());  
        var anon2 = new { X = 2 };  
        anon2 = anon1;         
        Console.WriteLine(anon1.X); // 1  
    }  
}

As you can see, you can share instances of an anonymous type around an assembly if you want to. We do not expect that a lot of people will do so; typically if this is the sort of thing you like doing, you'd use either a tuple or a custom-built nominal type. Still, it is not expensive for us to make this guarantee, and it is kind of a nice property.

-----

(\*) Technically not exactly true. You could have an assembly made up of many different netmodules compiled at different times that do not know about each other; anonymous types defined in one netmodule will not necessarily unify with those in others.

(\*\*) Next time I'll discuss why we have this latter requirement.

