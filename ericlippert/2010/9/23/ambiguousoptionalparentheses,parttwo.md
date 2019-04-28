# Ambiguous Optional Parentheses, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/23/2010 8:49:00 AM

-----

(﻿This is part two of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/20/ambiguous-optional-parentheses.aspx). Part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/27/ambiguous-optional-parentheses-part-three.aspx).)

Last time I discussed why in C\# 3.0 the design team decided to permit [elision](http://blogs.msdn.com/b/ericlippert/archive/2004/05/11/riddle-me-this-google.aspx) of the argument list on a constructor call that uses the object/collection initializer syntax. A natural follow-up question is:

> While you were in there anyway, why did you not make elision of *all* empty argument lists on constructor calls legal, even those that are not followed by an initializer?

Good question. Just to be clear, the feature being proposed here is to make the argument list optional in C\# 3.0. I am not discussing the question of "should this feature have been implemented in C\# 1.0?"; the fact is that it was *not* implemented in C\# 1.0, so the feature proposal has to be evaluated in that context.

Last time I gave some thoughts on the costs associated with a syntactic sugar, and noted that in order to make it into the product, the costs for a trivial sugar have to be really low. Perhaps surprisingly, the costs are much higher for this syntactic sugar than for the object initializer sugar. Why? Because **the proposed feature introduces an ambiguity in the semantic analysis phase of compilation of possibly-existing C\# 2.0 programs.**

class P  
{  
    class B  
    {  
        public class M { }  
    }  
    class C : B  
    {  
        new public void M(){}  
    }  
    static void Main()  
    {  
        new C().M(); // 1  
        new C.M();   // 2  
    }  
}  

In C\# 2.0, line 1 creates a new C, calls the default constructor, and then calls the instance method M on the new object. Line 2 creates a new instance of B.M and calls its default constructor. **If the parentheses on line 1 were optional then line 2 would be ambiguous.** We can come up with even more bizarre cases:

class P  
{     class B  
    {  
        public class M  
        {  
            public static implict operator D(M m) { return null; }  
        }  
    }  
    delegate D D();  
    class C : B  
    {  
        new public D M(){return null;}  
    }  
    static void Main()  
    {  
        D d = new C.M;  
    }  
}

This is not a legal C\# 2.0 program but it is deeply ambiguous if we allow the feature. There are three possibilities for the meaning of new C.M. If it is new C().M then it is a conversion of the method group M on an object of type C to the delegate type D. If it is new C.M() then that is the user-defined implicit conversion of an instance of B.M to D. And if it is new C().M() then it is the execution of method M on an instance of C, which returns a null D.

Yuck. We would have to come up with a rule resolving every possible ambiguity, possibly with a warning. (We would not make it an *error* because that might then be a breaking change that changes an existing legal C\# program into a broken program.)

The rule would have to be potentially rather complicated: essentially that the parentheses are only optional in cases where they don't introduce ambiguities. *Such a vague rule is of no use whatsoever to compiler writers.* We'd have to analyze all the possible cases that introduce ambiguities and then write code in the compiler to detect them.

In that light, go back and look at all the costs I mentioned last time. How many of them now become large? Complicated rules have relatively much larger design, spec, development, testing and documentation costs. Complicated rules are much more likely to cause problems with unexpected interactions with features in the future. They are more likely to cause performance or correctness problems in IDE features like IntelliSense.

All for what? A tiny-to-the-point-of-non-existant customer benefit that adds no new representational power to the language, but does add crazy corner cases just waiting to yell "gotcha" at some poor unsuspecting soul who runs into it.

Features like that get cut *immediately* and put on the "never do this" list.

> Holy goodness\! How did you find that ambiguity?

Next time\!

(﻿This is part two of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/20/ambiguous-optional-parentheses.aspx). Part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/27/ambiguous-optional-parentheses-part-three.aspx).)

﻿

