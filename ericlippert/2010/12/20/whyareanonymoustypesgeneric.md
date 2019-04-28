# Why are anonymous types generic?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/20/2010 6:30:00 AM

-----

Suppose you use an anonymous type in C\#:

 

var x = new { A = "hello", B = 123.456 };

Ever taken a look at what code is generated for that thing? If you crack open the assembly with ILDASM or some other tool, you'll see this mess in the top-level type definitions

 

.class '\<\>f\_\_AnonymousType0\`2'\<'\<A\>j\_\_TPar','\<B\>j\_\_TPar'\>

**What the heck?** Let's clean that up a bit. We've mangled the names so that you are guaranteed that you cannot possibly accidentally use this thing "as is" from C\#. Turning the mangled names back into regular names, and giving you the declaration and some of the body of the class in C\#, that would look like:

 

\[CompilerGenerated\]  
internal sealed class Anon0\<TA, TB\>  
{  
    private readonly TA a;  
    private readonly TB b;  
    public TA A { get { return this.a; } }  
    public TB B { get { return this.b; } }     
    public Anon0(TA a, TB b)  
    { this.a = a; this.b = b; }  
    // plus implementations of Equals, GetHashCode and ToString  
}

And then at the usage site, that is compiled as:

 

var x = new Anon0\<string, double\>("hello", 123.456);

Again, **what the heck?** Why isn't this generated as something perfectly straightforward, like:

 

\[CompilerGenerated\]  
internal sealed class Anon0  
{  
    private readonly string a;  
    private readonly double b;  
    public string A { get { return this.a; } }  
    public double B { get { return this.b; } }     
    public Anon0(string a, double b)  
    { this.a = a; this.b = b; }  
    // plus implementations of Equals, GetHashCode and ToString  
}

Good question. Consider the following.

Suppose you have a library assembly, not written by you, that contains the following types:

 

public class B  
{  
    protected class P {}  
}

Now, in your source code you have:

 

class D1 : B  
{  
    void M() { var x = new { P = new B.P() }; }  
}  
  
class D2 : B  
{  
    void M() { var x = new { P = new B.P() }; }  
}

We need to generate an anonymous type, or types, somewhere. Suppose we decide that we want the two anonymous types - which have the same types and the same property names - to unify into one type. (We desire anonymous types that are structurally identical to unify within an assembly because that enables scenarios where multiple methods use generic type inference to infer the same anonymous type; you want to be able to pass instances of that anonymous type around between such methods. Perhaps I'll do an example of that in the new year.)

Where do we generate that type? How about inside D1:

 

class D1 : B  
{  
    \[CompilerGenerated\]  
    ??? sealed class Anon0 { public P P { get { ... } } ... }  
    void M() { var x = new { P = new B.P() }; }  
}

What is the desired accessibilty of Anon0? It cannot be private or protected, because then D2 cannot see it. It cannot be either public or internal, because then you'd have a public/internal type with a public property that exposes a protected type, which is illegal. (Nor can it be either "protected and internal" or "protected or internal" by similar logic.) **It cannot have *any* accessibility\!** Therefore the anonymous type cannot go in D1.  Obviously by identical logic it cannot go in D2. It cannot go in B; it's just an assembly. The only remaining place it can go is in the global namespace. But at the top level an internal type cannot refer to P, a protected type. P is only accessible *inside* a derived class of B.

But we *can* put the anonymous type at the top level *if it never actually refers to P.* If we make generic class Anon0\<TP\> and *construct* it with P for TP, then P only ever appears inside D1 and D2, and yet the types unify as desired.

Rather than coming up with some weird heuristic that determined when anonymous types *needed* to be generic, and making them normally typed otherwise, we simply decided to embrace the general solution. Anonymous types are *always* generated as generic types even when doing so is not strictly necessary. We did extensive performance testing to ensure that this choice did not adversely impact realistic scenarios, and as it turned out, the CLR is really quite buff when it comes to construction of generic types with lots of type parameters.

And with that, I'm off for the rest of the year. Air travel is too expensive this year, so I'm going to miss my traditional family Boxing Day celebration, but I'm sure it'll be delightful to spend some time in Seattle for the holidays. I hope you all have a safe and festive holiday season, and we'll see you for more fabulous adventures in 2011.

