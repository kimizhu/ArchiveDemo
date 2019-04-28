# Reserved and Contextual Keywords

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/11/2009 8:07:00 AM

-----

Many programming languages, C\# included, treat certain sequences of letters as “special”.

Some sequences are so special that they cannot be used as identifiers. Let’s call those the “reserved keywords” and the remaining special sequences we’ll call the “contextual keywords”. They are “contextual” because the character sequence might one meaning in a context where the keyword is expected and another in a context where an identifier is expected.\*

The C\# specification defines the following reserved keywords:

 

abstract as base bool break byte case catch char checked class const continue decimal default delegate do double else enum event explicit extern false finally fixed float for foreach goto if implicit in int interface internal is lock long namespace new null object operator out override params private protected public readonly ref return sbyte sealed short sizeof stackalloc static string struct switch this throw true try typeof uint ulong unchecked unsafe ushort using virtual void volatile while

The implementation also reserves the magic keywords \_\_arglist \_\_makeref \_\_reftype \_\_refvalue which are for obscure scenarios that I might blog about in the future.

Those are the keywords that we reserved in C\# 1.0; no new reserved keywords have been added since. It is tempting to do so, but we always resist. Were we to add a new reserved keyword then any program that used that keyword as an identifier would break upon recompilation. Yes, you can always use a keyword as an identifier if you really want: @typeof @goto = @for.@switch(@throw); is perfectly legal, though more than a little weird. But we prefer to avoid as many breaking changes as possible.

We also have a whole bunch of contextual keywords.

The “preprocessor” † uses all the directives (\#define, and so on) which of course were never valid identifiers in the first place. But it also uses contextual keywords hidden default disable restore checksum.

C\# 1.0 had contextual keywords get set value add remove for properties, indexers and events. The attribute locations event and return are already reserved keywords; assembly module type method field property param typevar are contextual keywords in the context of an attribute.

C\# 2.0 added where partial global yield alias.

C\# 3.0 added from join on equals into orderby ascending descending group by select let var.

C\# 4.0 added dynamic.

The async CTP added async and await.

Every time we add one of these we need to carefully design the grammar so that if possible, the use of the new contextual keyword does not possibly change the meaning of an existing program which used it.

For example, when defining a partial class, the partial must go immediately before the class. Since there was never a legal C\# 1.0 program where partial appeared immediately before class, we knew that adding this new feature to the grammar would not possibly break any existing programs.

Or, another example. Consider var x = 1; – that could have been a legal C\# 2.0 program if there was a type called var with a user-defined implicit conversion from int. The semantic analyzer for declaration statements checks to see whether there is a type called var that is accessible at the declaration; if there is then the normal declaration rules are used. Only if there is not such a type can we do the analysis as an implicitly typed local declaration.

One might wonder why on earth we added five contextual keywords to C\# 1.0, when there was no chance of breaking backwards compatibility. Why not just make get set value add remove into “real” keywords?

Because we could easily get away with making them contextual keywords, and it seemed likely that real people would want to name variables or methods things like get, set, value, add or remove. So we left them unreserved as a courtesy.

Those were easy to make contextual, unlike, say, return. That’s a lot harder to make a contextual keyword because then return (10); would be ambiguous; is that calling the method named “return” or returning ten? So we didn’t make any of the other reserved keywords into contextual keywords.

\*\*\*\*\*\*\*

(\*) An unfortunate consequence of this definition is that using is said to be a reserved keyword even though its meaning depends on its context; whether the using begins a directive or a statement determines its meaning.

(†) An unfortunate name, since “preprocessing” is not done before regular language processing. In C\#, the so-called “preprocessing” happens during lexical analysis.

