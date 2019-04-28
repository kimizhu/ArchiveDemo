# Every Program There Is, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/3/2010 7:11:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/29/every-program-there-is-part-two.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/06/every-program-there-is-part-four.aspx).\]

Suppose we want to write a grammar for a simplified C\# class declaration. Let’s say that there are one-character identifiers, a class can be public or internal, and that there are no nested or generic types or other members. How about this?

 

DECL:     MODIFIER class ID : ID { }  
MODIFIER: public | internal  
ID:       a | b | c | d

so public class d : b { } is legal. But we forgot that the modifier should be optional:

 

DECL:           WITHMODIFIER | WITHOUTMODIFIER  
WITHMODIFIER:   MODIFIER WITHOUTMODIFIER  
WITHOUTMODIFER: class ID : ID { }  
MODIFIER:       public | internal  
ID:             a | b | c | d

And we forgot that the base class is optional…

 

DECL:            WITHMODIFIER | WITHOUTMODIFIER  
WITHMODIFIER:    MODIFIER WITHOUTMODIFIER  
WITHOUTMODIFIER: MAYBEBASE { }  
MAYBEBASE :      WITHOUTBASE : ID | WITHOUTBASE  
WITHOUTBASE:     class ID  
MODIFIER:        public | internal  
ID:              a | b | c | d

It’s doable, but as you can see, this is getting to be a bit of a mess. It is a lot easier if we can say that “nothing at all” is a legal substitution. Since that’s hard to write, I’m going to make a new rule that NIL is a special **terminal** that is a nice way of writing the empty string. Now our grammar can be much easier to read:

 

DECL:     MODIFIER class ID BASE { }  
MODIFIER: NIL | public | internal  
BASE:     NIL | : ID  
ID:       a | b | c | d

This addition has interesting consequences. Previously we were in the situation where every application of a rule made the "current string" no shorter in terms of the number of terminals plus the number of nonterminals. Now we have a situation where a substitution can make the resulting string have fewer nonterminals without adding any terminals. But I'm getting ahead of myself somewhat.

Here I've characterized "NIL" as a terminal which is a special way of writing the empty string. Another way of characterizing NIL is as a nonterminal; NIL as a nonterminal is a special nonterminal such that the only substitution is the empty string. For the rest of this series I'll treat it as a convenient way of writing the empty string terminal.

**Next time:** techniques for generating all members of a language

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/29/every-program-there-is-part-two.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/06/every-program-there-is-part-four.aspx).\]

