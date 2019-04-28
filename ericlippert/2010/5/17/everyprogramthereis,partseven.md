# Every Program There Is, Part Seven

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/17/2010 6:20:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/13/every-program-there-is-part-six.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

Things seem to be going so well. We have a sketch of a recursive solution for enumerating all strings of a particular grammar and it looks like the recursion is well-founded. Let’s consider one of our more complex grammars from a previous posting:

 

DECL:     MODIFIER class ID BASE { }  
MODIFIER: NIL | public | internal  
BASE:     NIL | : ID  
ID:       a | b | c | d

Consider the smallest string in this language class a { }. It’s derivation is in DECL\[5\]:

 

DECL  
MODIFIER class ID BASE { }       
NIL class ID BASE { }  
class ID BASE { }  
class a BASE { }  
class a { }

(Remember that we are characterizing NIL as being just a fancy way of writing the empty string. Here we substitute NIL for BASE and it just goes away.)

How would we work out DECL\[5\] using our algorithm? Easy: just solve all these subproblems and combine the solutions:

 

MODIFIER\[0\] class\[0\] ID\[0\] BASE\[0\] {\[0\] }\[4\]  
MODIFIER\[0\] class\[0\] ID\[0\] BASE\[0\] {\[1\] }\[3\]  
MODIFIER\[0\] class\[0\] ID\[0\] BASE\[0\] {\[2\] }\[2\]  
MODIFIER\[0\] class\[0\] ID\[0\] BASE\[0\] {\[3\] }\[1\]  
… hundreds more…  
MODIFIER\[3\] class\[1\] ID\[0\] BASE\[0\] {\[0\] }\[0\]  
MODIFIER\[4\] class\[0\] ID\[0\] BASE\[0\] {\[0\] }\[0\]

Um… yuck. Even if we write optimizations in there so that we skip even considering X\[0\] on nonterminals and X\[n\>0\] on terminals, it is still a total pain in the rear to write the code to do this. We didn’t have this problem in our earlier example because we only ever had one or two terms for a given substitution. If only we could have the simplicity of having only two terms per production.

Hmm.

Remember back when we had the grammar

S: N | S + N  
N: 1 | 2 | 3

and we noticed that this was the same language as the grammar

S: N | S P  
P: + N  
N: 1 | 2 | 3

? Notice that what we did was we turned a grammar where there was a rule with three terms into a grammar where every rule is either one or two terms. Now, what if we also did this?

S: N NIL | S P  
P: + N  
N: 1 NIL | 2 NIL | 3 NIL

That’s clearly also the same language. **We can take any grammar and turn it into an equivalent grammar that has exactly two terms per substitution.** Either we add a NIL to a “single” production, or we make an n-ary production into an n-1-ary production by creating a new substitution rule.

Here’s an unambiguous grammar that lets us define public generic classes with base types and nested classes:

 

CLASSLIST:       CLASSDECL CLASSLIST | NIL  
CLASSDECL:       public class ID PARAMSLIST BASE { CLASSLIST }  
PARAMSLIST:      NIL | \< PARAMSLISTBODY \>  
PARAMSLISTBODY:  ID | ID , PARAMSLISTBODY  
BASE:            NIL | : TYPE  
TYPE:            ID TYPEARGS | ID TYPEARGS . TYPE  
TYPEARGS:        NIL | \< ARGLISTBODY \>  
ARGLISTBODY:     TYPE  | TYPE , ARGLISTBODY  
ID:              a | b | c | d

We can rewrite this into one of our special grammars that have two terms per production like this:  

CLASSLIST:       CLASSDECL CLASSLIST | NIL NIL  
CLASSDECL:       HEADER CLASSBODY  
HEADER:          CLSNAME BASE  
CLSNAME:         PUBCLS CLASSNAME  
PUBCLS:          public class  
CLASSNAME:       ID PARAMSLIST  
PARAMSLIST:      NIL NIL | \< PARAMSLISTEND  
PARAMSLISTEND:   PARAMSLISTBODY \>  
PARAMSLISTBODY:  ID PARAMSLISTREST  
PARAMSLISTREST:  NIL NIL | , PARAMSLISTBODY  
BASE:            NIL NIL | : TYPE  
TYPE:            NODOTTYPE NIL | TYPEDOT TYPE  
NODOTTYPE:       ID TYPEARGS  
TYPEDOT:         NODOTTYPE .  
TYPEARGS:        NIL NIL | \< ARGLISTEND  
ARGLISTEND:      ARGLISTBODY \>  
ARGLISTBODY:     TYPE ARGLISTREST  
ARGLISTREST:     NIL NIL | , ARGLISTBODY  
CLASSBODY:       { CLASSBODYEND  
CLASSBODYEND:    CLASSLIST }  
ID:              a NIL | b NIL | c NIL | d NIL 

Harder to read of course, but defining the same language. And in fact, we could write a program that takes a grammar of the first form and automatically produces a grammar of the second form, though we’re not going to do that.

**Next time:** Looks like we’re actually ready to start writing code\!

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/13/every-program-there-is-part-six.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

