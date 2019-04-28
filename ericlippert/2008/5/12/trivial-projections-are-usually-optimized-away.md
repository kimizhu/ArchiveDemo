<div id="page">

# Trivial Projections Are (Usually) Optimized Away

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/12/2008 9:57:00 AM

-----

<div id="content">

<div class="mine">

OK, computers aren't *entirely* dumb when it comes to LINQ. Here's an example of a place where we're a bit smarter.

Consider the following query:

<span class="code"> </span>

IEnumerable\<int\> query = from n in number\_array orderby n select n;

Does this get transformed by the compiler into

<span class="code"> </span>

IEnumerable\<int\> query = number\_array.OrderBy(n =\> n);

or

<span class="code"> </span>

IEnumerable\<int\> query = number\_array.OrderBy(n =\> n).Select( n =\> n );

?

This question caused tremendous debate amongst the members of the C\# design and implementation teams. I am quite happy with the compromise we achieved.

The sections of the C\# specification which cover this topic are sections 7.15.2.3, and 7.15.2.5:

\*\*\*

<span class="spec"> </span>

A query expression of the form “from x in e select v” is translated into “( e ) . Select ( x =\> v )” except when v is the identifier x, the translation is simply “( e )”

A query expression of the form “from x in e select x” is translated into “( e ) . Select ( x =\> x )”.

A degenerate query expression is one that trivially selects the elements of the source. A later phase of the translation removes degenerate queries introduced by other translation steps by replacing them with their source. It is important however to ensure that the result of a query expression is never the source object itself, as that would reveal the type and identity of the source to the client of the query. Therefore this step protects degenerate queries written directly in source code by explicitly calling Select on the source. It is then up to the implementers of Select and other query operators to ensure that these methods never return the source object itself.

\*\*\*

It might sound like the second statement contradicts the first. The first statement says that the “<span class="code">.Select(x=\>v)</span>” is omitted when “<span class="code">v</span>” is “<span class="code">x</span>”, and the second statement says that no, the “<span class="code">.Select(x=\>x)</span>” is in fact generated.

The spec is not particularly clear on this point, I agree, though the explanatory text about “degenerate query expressions” does help clear it up somewhat. Basically what we are saying here is that the optimization to remove degenerate selects is only performed on the result of an earlier query translation.

Some examples will help.

If you have <span class="code">from n in number\_array orderby n select n</span> then this is first translated into <span class="code">from n in (number\_array).OrderBy(n=\>n) select n </span>and then, since the selector is the same as the range variable, this is translated into <span class="code">((number\_array).OrderBy(n=\>n))</span>.

We optimize away the <span class="code">Select(n=\>n)</span> because <span class="code">from n in (number\_array).OrderBy(n=\>n) select n </span>is the intermediate result of an earlier query translation.

If you had <span class="code">from n in number\_array select n </span>then this would be considered a degenerate query expression requiring a <span class="code">Select</span>. We cannot simply translate it into <span class="code">(number\_array)</span> because then you could assign that to a variable and have referential identity with the collection. The result of the query should not “leak” information about the collection out; it should just be an enumerable object of the appropriate type, and not have an identity relationship. Therefore, we take the performance hit and protect the identity of the collection by emitting a degenerate query translation <span class="code">(number\_array).Select(n=\>n)</span>.

This is important because the collection might be a mutable collection. One does not expect that the result of a *query* over that collection could be used to mutate the collection\! The query results should be read-only.

We do not need to append the degenerate <span class="code">Select(n=\>n)</span> when you have an “<span class="code">orderby”</span> in there because the <span class="code">OrderBy(n=\>n)</span> call already protects the identity of the original collection. This saves on the performance cost of the no-op <span class="code">Select</span>.

I think we’ve made a reasonable compromise in the case of degenerate queries. If you are crazy enough to write “<span class="code">from x in xs select x</span>” when you mean “<span class="code">(xs)</span>”, then you’ll have to live with the decreased performance. But if you are introducing a degenerate select at the end of a meaningful query, then it will be optimized away.

</div>

</div>

</div>

