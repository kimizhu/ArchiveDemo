# Scripting Type Library Constant Injection Performance Characteristics, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/22/2005 10:00:00 AM

-----

(Sorry about the title. I work for Microsoft; [we like nouns](http://blogs.msdn.com/ericlippert/archive/2005/06/06/425847.aspx).) Over a year ago now a reader noted in a comment to [this posting](http://blogs.msdn.com/ericlippert/archive/2004/06/01/145686.aspx) that defining named constants using Const in VBScript or var in JScript is way, way faster than importing a type library. My empirical JScript testing showed that loading constants from a typelib is one or two orders of magnitude slower than preloading an engine with script which defines those same constants. I responded I wrote all the type library importation code -- you are correct, it is NOT FAST compared to simply defining a bunch of consts/vars. There are good reasons why. I'll blog about them at some point. At long last I'll get around to explaining why. There are interesting design tradeoffs here, and as often happens in the scripting world, performance was traded off against ease-of-use. We're built for comfort, not speed. [As I have said many times before, if you care about maximizing performance, using a late-bound unoptimized bytecode-interpreted dynamically-typed language is probably a bad choice.](http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx) Let's go through the design process first, and then get into the implementation details and what the performance costs are. Let's make up a little syntax to describe the type libraries that I'm talking about. Note that this is **not VBScript**, I just need a concise way to describe the enumerated type contents of a type library. (String constants in modules work the same way and nothing else in the library is relevant, so without loss of generality I'll be considering only enumerated types in this series.) I'll make it a bracy language to emphasize that this is not VBScript. typelib Video {  
  enum Color {  
    Red = 1  
    Green = 2  
    Blue = 3  
    Black = 4  
    White = 5  
  }  
} typelib Food {  
  enum Cheese {  
    Green = 1  
    Blue = 2  
    Yellow = 3  
    White = 4  
  }  
} What does this VBScript program do if those type libraries have been added to the script engine namespace? foo.bar = Blue We've got a bit of a collision problem here. An obvious technique to solve this problem is to allow qualification: foo.bar = Color.Blue ' OK, it's 3 Which works great until we also add this type library: typelib Magic {  
  enum Color {  
    White = 1  
    Black = 2  
  }  
} Now it's not clear what foo.bar = Color.White means. We need two levels of disambiguation: foo.bar = Video.Color.White ' five Our major design decision here is to decide whether full qualification is a requirement **all the time**, or **only when there is a collision**. Or, somewhere in between; C\# requires that enumerations be qualified by at least the type name but makes explicit full qualification optional via the "using" statement. C doesn't allow qualification at all -- you get a collision, that's your problem. (This is why you see so many enums in C that look like enum MagicColor { magicColorWhite, magicColorBlack }; in an attempt to avoid collisions. We gave this a lot of thought and decided that the likely scenario was that collisions would be rare. Therefore users should not be forced to always qualify their enumerations at either the enumeration or library level. However, since collisions *might* occur, we should not act like C. In short, **full, partial and no qualification should all be legal**. There are other design constraints that might not be immediately obvious. For example, **full qualification must always work even in a world where type information is not known at compile time**. Suppose you have the three type libraries above plus typelib Cheese {  
  enum Blue{  
    Stilton = 1  
    Gorgonzola = 2  
  }  
} The code foo.bar = Cheese.Blue.Stilton must succeed -- we can't decide at runtime that this really means the nonsensical Food.Cheese.Blue.Stilton and raise an error. But we cannot do any analysis before the code runs either, because for all we know, someone used Execute to introduce a local object variable called Cheese, which would then win over either of those options. Furthermore, we might not even have the type libraries available when the compiler runs: the host can choose to add them after the script has been compiled. (Though that would be irksome, it is legal.) We can't make any assumptions at code generation time -- we have to assume that this is a series of object property calls like any other. Next time we'll revisit how the script engines handle host-supplied objects, and see how that mechanism neatly solves our design problem. Then we'll see why this isn't fast compared to declaring the constant yourself.

