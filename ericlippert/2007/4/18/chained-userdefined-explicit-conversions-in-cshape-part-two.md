<div id="page">

# Chained user-defined explicit conversions in C\#, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/18/2007 10:00:00 AM

-----

<div id="content">

<div class="mine">

Reader Larry Lard asks a follow-up question regarding [the subject of Monday’s blog entry](http://blogs.msdn.com/ericlippert/archive/2007/04/16/chained-user-defined-explicit-conversions-in-c.aspx). Why is it that the compiler knows that <span class="code">(int)(new Base())</span> will always fail, and therefore makes the conversion illegal, but does not know that <span class="code">(Derived)(new Base())</span> will always fail, and make that conversion illegal too?

There are two answers, a general answer and a specific answer.

The general answer is that in most cases, the C\# type system operates solely on types. There are some exceptions where we peer closer at an expression when doing a conversion – lambdas are the obvious huge exception to that rule, since [the convertibility of a lambda to a particular type depends on the structure of the type and the contents of the lambda parameters and body](http://blogs.msdn.com/ericlippert/archive/2007/01/10/lambda-expressions-vs-anonymous-methods-part-one.aspx). There are smaller exceptions as well – [literal zero is convertible to any enum](http://blogs.msdn.com/ericlippert/archive/2006/03/28/563282.aspx), for example. But in general, we cleave to the principle that the type system mostly makes decisions based on the *types* of expressions, not the *content* of the expressions themselves.

Thus, in the examples above, we only see that the argument of the cast operator is something of type <span class="code">Base</span>. The fact that this thing can be no more derived than <span class="code">Base</span> is lost; it’s something of type <span class="code">Base</span>, and therefore might be something that can be converted to <span class="code">Derived</span>.

The more specific answer is that **in fact, this doesn’t always fail.** Betcha didn’t know that\!

This is one of the most obscure, bizarre (and frankly, also one of the most poorly specified and documented) parts of the C\# implementation. I myself just happened to learn about it recently. But it is true -- there is a situation where there is a non-user-defined explicit conversion from <span class="code">Base</span> to <span class="code">Derived</span> such that <span class="code">(Derived)(new Base())</span> succeeds at runtime.

Via a similar mechanism, there is also a situation where <span class="code">(Base)(new Base())</span> fails at runtime\!

Actually, it gets even worse than that. Last time, I mentioned that there were two times when the compiler inserts an "explicit" cast on your behalf. The case I am referring to here introduces a third that I didn't know about. This means that it is possible for <span class="code">Base b = new Base();</span> to compile but fail at runtime\!

So, another challenge to my readers: does anyone know the extremely obscure way that this can happen?

I’ll give you a hint: the <span class="code">Base</span> will typically start with the letter <span class="code">I</span>, not <span class="code">B</span>...

And in related news, I've also recently learned of a *fourth* situation in which the compiler inserts an explicit cast. I'll blog more about that later this week probably.

</div>

</div>

</div>

