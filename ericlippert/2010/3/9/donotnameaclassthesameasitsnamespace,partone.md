# Do not name a class the same as its namespace, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/9/2010 6:52:00 AM

-----

(This is part one of a four part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/11/do-not-name-a-class-the-same-as-its-namespace-part-two.aspx).)

The Framework Design Guidelines say in section 3.4 “do not use the same name for a namespace and a type in that namespace”. (\*) That is:

 

namespace MyContainers.List  
{  
    public class List { … }  
}

Why is this badness? Oh, let me count the ways.

**Part One: Collisions amongst referenced assemblies:**

You can get yourself into situations where you think you are referring to one thing but in fact are referring to something else. Suppose you end up in this unfortunate situation: you are writing Blah.DLL and importing Foo.DLL and Bar.DLL, which, unfortunately, both have a type called Foo:

 

// Foo.DLL:  
namespace Foo { public class Foo { } }

// Bar.DLL:  
namespace Bar { public class Foo { } }

// Blah.DLL:  
namespace Blah  
{  
  using Foo;  
  using Bar;  
  class C { Foo foo; }  
}

The compiler gives an error. “Foo” is ambiguous between Foo.Foo and Bar.Foo. Bummer. I guess I’ll fix that by fully qualifying the name:

  class C { Foo.Foo foo; }  

This now gives the ambiguity error “Foo in Foo.Foo is ambiguous between Foo.Foo and Bar.Foo”. We still don’t know what the first Foo refers to, and until we can figure that out, we don’t even bother to try to figure out what the second one refers to.

This reveals an interesting point about the design of the “type binding” algorithm in C\#. That is, the algorithm which determines what type or namespace a name like “X.Y” is talking about. We do not “backtrack”. We do not say “well, suppose X means this. Then Y would have no meaning. Let’s backtrack; suppose X means this other thing, oh, yes, then Y has a meaning.” We figure out what X unambiguously means, and only then do we figure out what Y means. If X is ambiguous, we don’t check all the possibilities to see if any of them has a Y, we just give up.

Assuming you cannot change Foo.DLL, the correct thing to do here is to either remove the “using Foo” – and who knows what all that will break – or to use an extern alias when compiling:

 

// Blah.DLL:  
extern alias FOODLL;  
namespace Blah  
{  
  using Foo;  
  using Bar;  
  class C { FOODLL::Foo.Foo foo; }  
}

Many developers are unfamiliar with the “extern alias” feature. And many of those developers who end up in this situation thereby end up in a cleft stick not of their own devising. Some of them send an angry and profane email to the person who did not cause the problem in the first place, namely, me.

The problem can be avoided in the first place by the authors of Foo.DLL following the guidelines and not naming their type and their namespace the same thing.

Next time: machine-generated code throws a wrench into its own works.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) FXCOP flags violations of a far more stringent guideline – FXCOP will tell you if \*any\* type name conflicts with \*any\* namespace. But today I’m just talking about the straight-up type-in-a-namespace scenario.

(This is part one of a four part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/11/do-not-name-a-class-the-same-as-its-namespace-part-two.aspx).)

