# Optional argument corner cases, part four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/19/2011 8:19:00 AM

-----

(This is the fourth and final part of a series on the corner cases of optional arguments in C\# 4; part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/16/optional-argument-corner-cases-part-three.aspx).)

Last time we discussed how some people think that an optional argument generates a bunch of overloads that call each other. People also sometimes incorrectly think that

 

void M(string format, bool b = false)  
{  
  Console.WriteLine(format, b);  
}

is actually a syntactic sugar for something morally like:

 

void M(string format, bool? b)  
{  
  bool realB = b ?? false;  
  Console.WriteLine(format, realB);  
}

and then the call site

 

M("{0}");

is rewritten as

 

M("{0}", null};

That is, they believe that the default value is somehow "baked in" to the **callee**.

In fact, the default value is baked in to the **caller**; the code on the callee side is untouched and the caller becomes

 

M("{0}", false);

A consequence of this fact is that if you change the default value of a library method without recompiling the callers of that library, the callers don't change their behaviour just because the default changed. If you ship a new version of method "M" that changes the default to "true" it doesn't matter to those callers. Until a caller of M with one argument is recompiled it will always pass "false".

That could be a good thing. Changing a default from "false" to "true" is a breaking change, and one could argue that existing callers \*should\* be insulated from that breaking change.

This is a fairly serious versioning issue, and one of the main reasons why we pushed back for so long on adding default arguments to C\#. The lesson here is to think carefully about the scenario with the long term in mind. If you suspect that you will be changing a default value and you want the callers to pick up the change without recompilation, don't use a default value in the argument list; make two overloads, where the one with fewer parameters calls the other.

(This is the fourth and final part of a series on the corner cases of optional arguments in C\# 4; part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/16/optional-argument-corner-cases-part-three.aspx).)

