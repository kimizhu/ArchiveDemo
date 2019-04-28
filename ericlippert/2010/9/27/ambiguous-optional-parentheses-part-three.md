<div id="page">

# Ambiguous Optional Parentheses, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/27/2010 10:08:00 AM

-----

<div id="content">

<div class="mine">

(﻿This is part three of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/23/ambiguous-optional-parentheses-part-two.aspx).)

Last time I discussed why a particular syntactic sugar would have been rejected by the design team: because it introduces the costs of a nasty semantic analysis ambiguity without any corresponding benefit. Continuing on with our dialogue...

> <span style="color: #0000ff;">How did you determine that particular ambiguity?</span>

I am pretty familiar with the rules in C\# for determining when a dotted name is expected and how the semantic analyzer resolves it. A few minutes double-checking the spec and trying out some sample programs in the compiler confirmed my initial suspicions.

> <span style="color: #0000ff;">Fair enough. But when considering a new feature how do you in general determine whether it causes any ambiguity? By hand? by formal proof? by machine analysis? what?</span>

All three. Mostly we just look at the spec and noodle on it, as I did for the proposed feature last time.

Let me take you through an example of how we might noodle around the spec looking for ambiguities. Suppose hypothetically we wanted to add a new **prefix unary operator** to C\#, called <span class="code">frob</span>:

(**UPDATE**: I can now tell you that this was the exact analysis we went through when adding "async" to the language; these are all cases we considered when adding that prefix unary operator.)

<span class="code"> </span>

x = frob 123 + 456

<span class="code">frob</span> here is like <span class="code">new</span> or <span class="code">++</span> - it comes before an expression of some sort.

First thing we'd work out is the desired precedence and associativity and so on, so that we know whether this means <span class="code">frob(123+456)</span> or <span class="code">(frob 123)+456</span>

Then we'd start asking questions like "what if the program already has a type, field, property, event, method, constant, or local called frob?" That would immediately lead to cases like:

<span class="code"> </span>

frob x = 10;

does that mean "do the frob operation on the result of <span class="code">x = 10</span>, or create a variable of type <span class="code">frob</span> called x and assign 10 to it?" Or consider this case:

<span class="code"> </span>

G(frob + x);

Does that mean "frob the result of the unary plus operator on x" or "add expression frob to x"?

And so on. To resolve these ambiguities we might introduce heuristics. We've seen these sorts of problems before, after all. When you say <span class="code">var x = 10;</span> that's ambiguous; it could mean "infer the type of x" or it could mean "x is of type var". So we have a heuristic: we first attempt to look up a type named "var", and only if one does not exist do we infer the type of x.

Or, we might change the syntax so that it is not ambiguous. When they designed C\# 2.0 they had this problem:

<span class="code"> </span>

yield(x);

Does that mean "yield x in an iterator" or "call the yield method with argument x?" By changing it to

<span class="code"> </span>

yield return(x);

it is now unambiguous; it cannot possibly mean "call the yield method".

In the case of optional parens in an object initializer argument list it is straightforward to reason about whether there are ambiguities introduced or not. Why? Because the argument initializer has a left curly brace after the constructor. *The number of situations in which it is permissible to introduce something that starts with a left curly brace is very small*. Basically just various statement contexts, statement lambdas, array initializers and that's about it. It's easy to reason through all the cases and show that there's no ambiguity. Making sure the IDE stays efficient is somewhat harder (because you have to deal with partially-edited programs that might have braces in strange places) but can be done without too much trouble.

This sort of fiddling around with the spec usually is sufficient. If it is a particularly tricky feature then we pull out heavier-duty tools. For example, when designing LINQ, one of the compiler guys and one of the IDE guys who both have a background in parser theory built themselves a parser generator that could analyze grammars looking for ambiguities, and then fed proposed C\# grammars for query comprehensions into it; doing so found many cases where queries were ambiguous.

If a feature is really quite tricky and we want to get a professional opinion on it, we'll call in the super geniuses. For example, when we did advanced type inference on lambdas in C\# 3.0 we wrote up our proposals and then sent them over the pond to Microsoft Research in Cambridge where the languages team there was good enough to work up a formal proof that the type inference proposal was theoretically sound.

(﻿This is part three of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/23/ambiguous-optional-parentheses-part-two.aspx).) 

<span></span>

</div>

</div>

</div>

