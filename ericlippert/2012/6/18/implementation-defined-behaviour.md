<div id="page">

# Implementation-defined behaviour

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/18/2012 10:30:03 AM

-----

<div id="content">

<div class="mine">

As I've mentioned several times on this blog [before](http://blogs.msdn.com/b/ericlippert/archive/2007/08/14/c-and-the-pit-of-despair.aspx), C\# has been carefully designed to eliminate some of the "undefined behaviour" and "implementation-defined behaviour" that you see in languages like C and C++. But I'm getting ahead of myself; I should probably start by making a few definitions.

Traditionally we say that a programming language idiom has **undefined behaviour** if use of that idiom can have any effect whatsoever; it can work the way you expect it to or it can erase your hard disk or crash your machine. Moreover, the compiler author is under no obligation to warn you about the undefined behaviour. (And in fact, there are some languages in which programs that use "undefined behaviour" idioms are permitted by the language specification to crash the compiler\!) An example of undefined behaviour in C, C++ and C\# is writing to a dereferenced pointer that you have no business writing to. Doing so might cause a fault that crashes the process. But the memory could be valid, and it could be an important data structure of the runtime software. It could be code that you are now overwriting with code that does something else. And hence, anything can happen; you're rewriting the software that is running into software that does something else.

By contrast, an idiom that has **implementation-defined behaviour** is behaviour where the compiler author has several choices about how to implement the feature, and must choose one. As the name implies, implementation-defined behaviour is at least defined. For example, C\# permits an implementation to throw an exception or produce a value when an integer division overflows, but the implementation must pick one. It cannot erase your hard disk. All that said, for the rest of this article I'm not going to make a strong distinction between these two flavours of underspecified behaviour. The question I'm actually interested in addressing today is: **What are some of the factors that lead a language design committee to leave certain language idioms as undefined or implementation-defined behaviours?** The first major factor is: **are there two existing implementations of the language in the marketplace that disagree on the behaviour of a particular program?** If FooCorp's compiler compiles <span class="code">M(A(), B())</span> as "call A, call B, call M", and BarCorp's compiler compiles it as "call B, call A, call M", and neither is the "obviously correct" behaviour then there is strong incentive to the language design committee to say "you're both right", and make it implementation defined behaviour. *Particularly this is the case if FooCorp and BarCorp both have representatives on the committee.* The next major factor is: **does the feature naturally present many different possibilities for implementation, some of which are clearly better than others?** For example, in C\# the compiler's analysis of a "query comprehension" expression is specified as "do a syntactic transformation into an equivalent program that does not have query comprehensions, and then analyze that program normally". There is very little freedom for an implementation to do otherwise. For example, you and I both know that  

from c in customers  
from o in orders  
where c.Id == o.CustomerId  
select new {c, o} and  

from c in customers  
join o in orders on c.Id equals o.CustomerId  
select new {c, o} are semantically the same, and that the latter is likely to be more efficient. But the C\# compiler never, ever turns the first query expression syntax into a call to Join; it always turns it into calls to SelectMany and Where. The runtime implementation of those methods is of course fully within its rights to detect that the object returned by SelectMany is being passed to Where, and to build an optimized join if it sees fit, but the C\# compiler does not make any assumptions like that. You will always get a call to SelectMany out of the first syntax, and never a call to Join. We wanted the query comprehension transformation to be syntactic; the smart optimizations can be in the runtime. By contrast, the C\# specification says that the <span class="code">foreach</span> loop should be treated as the equivalent <span class="code">while</span> loop inside a <span class="code">try</span> block, but allows the implementation some flexibility. A C\# compiler is permitted to say, for example "I know how to implement the loop semantics more efficiently over an array" and use the array's indexing feature rather than converting the array to a sequence as the specification suggests it should. A C\# implementation is permitted to skip calling <span class="code">GetEnumerator</span>.

A third factor is: **is the feature so complex that a detailed breakdown of its exact behaviour would be difficult or expensive to specify?** The C\# specification says very little indeed about how anonymous methods, lambda expressions, expression trees, dynamic calls, iterator blocks and async blocks are to be implemented; it merely describes the desired semantics and some restrictions on behaviour, and leaves the rest up to the implementation. Different implementations could reasonably do different codegen here and still get good behaviours. A fourth factor is: **does the feature impose a high burden on the compiler to analyze?** For example, in C\# if you have:

Func\<int, int\> f1 = (int x)=\>x + 1;  
Func\<int, int\> f2 = (int x)=\>x + 1;  
bool b = object.ReferenceEquals(f1, f2);

Suppose we were to *require* b to be true. *How are you going to determine when two functions are "the same"*? Doing an "intensionality" analysis -- do the function bodies have the same content? -- is hard, and doing an "extensionality" analysis -- do the functions have the same results when given the same inputs? -- is even harder. A language specification committee should seek to minimize the number of open research problems that an implementation team has to solve\! In C\# this is therefore left to be implementation-defined; a compiler can choose to make them reference equal or not at its discretion. A fifth factor is: **does the feature impose a high burden on the runtime environment?** For example, in C\# dereferencing past the end of an array is well-defined; it produces an array-index-was-out-of-bounds exception. This feature can be implemented with a small -- not zero, but small -- cost at runtime. Calling an instance or virtual method with a null receiver is defined as producing a null-was-dereferenced exception; again, this can be implemented with a small, but non-zero cost. The benefit of eliminating the undefined behaviour pays for the small runtime cost. But the cost of determining if an arbitrary pointer in unsafe code is safe to dereference would have a large runtime cost, and so we do not do it; we move the burden of making that determination to the developer, who, after all, is the one who turned off the safety system in the first place. A sixth factor is: **does making the behaviour defined preclude some major optimization**? For example, C\# defines the ordering of side effects *when observed from the thread that causes the side effects*. But the behaviour of a program that observes side effects of one thread from another thread is implementation-defined except for a few "special" side effects. (Like a volatile write, or entering a lock.) If the C\# language required that all threads observe the same side effects in the same order then we would have to restrict modern processors from doing their jobs efficiently; modern processors depend on out-of-order execution and sophisticated caching strategies to obtain their high level of performance.

Those are just a few factors that come to mind; there are of course many, many other factors that language design committees debate before making a feature "implementation defined" or "undefined".

</div>

</div>

</div>

