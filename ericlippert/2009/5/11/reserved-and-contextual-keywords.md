<div id="page">

# Reserved and Contextual Keywords

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/11/2009 8:07:00 AM

-----

<div id="content">

<div class="mine">

Many programming languages, C\# included, treat certain sequences of letters as “special”.

Some sequences are so special that they cannot be used as identifiers. Let’s call those the “reserved keywords” and the remaining special sequences we’ll call the “contextual keywords”. They are “contextual” because the character sequence might one meaning in a context where the keyword is expected and another in a context where an identifier is expected.\*

The C\# specification defines the following reserved keywords:

<span class="code"> </span>

abstract as base bool break byte case catch char checked class const continue decimal default delegate do double else enum event explicit extern false finally fixed float for foreach goto if implicit in int interface internal is lock long namespace new null object operator out override params private protected public readonly ref return sbyte sealed short sizeof stackalloc static string struct switch this throw true try typeof uint ulong unchecked unsafe ushort using virtual void volatile while

The implementation also reserves the magic keywords <span class="code">\_\_arglist \_\_makeref \_\_reftype \_\_refvalue</span> which are for obscure scenarios that I might blog about in the future.

Those are the keywords that we reserved in C\# 1.0; no new reserved keywords have been added since. It is tempting to do so, but we always resist. Were we to add a new reserved keyword then any program that used that keyword as an identifier would break upon recompilation. Yes, you can always use a keyword as an identifier if you really want: <span class="code">@typeof @goto = @for.@switch(@throw);</span> is perfectly legal, though more than a little weird. But we prefer to avoid as many breaking changes as possible.

We also have a whole bunch of contextual keywords.

The “preprocessor” † uses all the directives (<span class="code">\#define</span>, and so on) which of course were never valid identifiers in the first place. But it also uses contextual keywords <span class="code">hidden default disable restore checksum</span>.

C\# 1.0 had contextual keywords <span class="code">get set value add remove</span> for properties, indexers and events. The attribute locations <span class="code">event</span> and <span class="code">return</span> are already reserved keywords; <span class="code">assembly module type method field property param typevar</span> are contextual keywords in the context of an attribute.

C\# 2.0 added <span class="code">where partial global yield alias</span>.

C\# 3.0 added <span class="code">from join on equals into orderby ascending descending group by select let var</span>.

C\# 4.0 added <span class="code">dynamic</span>.

The async CTP added <span class="code">async </span>and <span class="code">await</span>.

Every time we add one of these we need to carefully design the grammar so that if possible, the use of the new contextual keyword does not possibly change the meaning of an existing program which used it.

For example, when defining a partial class, the <span class="code">partial</span> must go immediately before the <span class="code">class</span>. Since there was never a legal C\# 1.0 program where <span class="code">partial</span> appeared immediately before <span class="code">class</span>, we knew that adding this new feature to the grammar would not possibly break any existing programs.

Or, another example. Consider <span class="code">var x = 1;</span> – that could have been a legal C\# 2.0 program if there was a type called var with a user-defined implicit conversion from int. The semantic analyzer for declaration statements checks to see whether there is a type called var that is accessible at the declaration; if there is then the normal declaration rules are used. Only if there is not such a type can we do the analysis as an implicitly typed local declaration.

One might wonder why on earth we added five contextual keywords to C\# 1.0, when there was no chance of breaking backwards compatibility. Why not just make <span class="code">get set value add remove</span> into “real” keywords?

Because we could easily get away with making them contextual keywords, and it seemed likely that real people would want to name variables or methods things like get, set, value, add or remove. So we left them unreserved as a courtesy.

Those were easy to make contextual, unlike, say, <span class="code">return</span>. That’s a lot harder to make a contextual keyword because then <span class="code">return (10);</span> would be ambiguous; is that calling the method named “return” or returning ten? So we didn’t make any of the other reserved keywords into contextual keywords.

\*\*\*\*\*\*\*

(\*) An unfortunate consequence of this definition is that <span class="code">using</span> is said to be a reserved keyword even though its meaning depends on its context; whether the <span class="code">using</span> begins a directive or a statement determines its meaning.

(†) An unfortunate name, since “preprocessing” is not done before regular language processing. In C\#, the so-called “preprocessing” happens during lexical analysis.

</div>

</div>

</div>

