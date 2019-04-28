<div id="page">

# Calling static methods on type parameters is illegal, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/18/2007 10:14:00 AM

-----

<div id="content">

<div class="mine">

[Last time](http://blogs.msdn.com/ericlippert/archive/2007/06/14/calling-static-methods-on-type-variables-is-illegal-part-one.aspx) I pointed out that static methods are always determined exactly at compile time, and used that fact to justify why static methods cannot be called on type parameter types. But aren’t the type arguments to generics actually determined at compile time?

On the caller side they are, sure. But on the callee side, the code emitted at compile time for a generic method is entirely generic. *It is not until the jitter encounters the code at runtime that the substitution of type arguments for type parameters is done.*

Consider a generic type:

<span class="code"> </span>

public class C\<T\> { public string M() { return typeof(T).ToString(); } }

When you compile that, the compiler emits a generic class definition which says just that – that we have a class, it has a type parameter, and a method which calls <span class="code">ToString()</span>. That is *all* the code which is emitted for this class at compile time.

Let me make sure that is clear. When you say

<span class="code"> </span>

void N() { C\<int\> c = new C\<int\>(); string s = c.M(); //...

the *compiler* **does not emit a copy of the class’s IL with <span class="code">int</span> substituted for <span class="code">T</span>**.

Rather, what happens is when your method <span class="code">N</span> is **jitted**, the jitter says hey, I need to jit up <span class="code">C\<int\>.M</span>. At that point the jitter consumes the generic IL emitted for <span class="code">C\<T\>.M</span> and creates brand new x86 (or whatever) code with <span class="code">int</span> substituted for <span class="code">T</span>.

Contrast this with C++ templates. C++ templates do not define generic types. Rather, C++ templates are basically a clever compile-time syntax for some complex search-and-replace macros. If you say <span class="code">C\<int\></span> in C++, then at compile time the C++ compiler textually substitutes <span class="code">int</span> for <span class="code">T</span> and emits code *as if that was how you had written it in the first place.*

If C\# had templates instead of macros then a static method called on a template parameter really *would* be determined at compile time, because the entire constructed class would be resolved at compile time. In this sense templates are a more powerful mechanism than generics – you can do crazy things with templates because *there is no type safety imposed upon the template as a whole*. Rather, the type safety is only checked for *every construction of the template actually in the program*.

But C\# generic types are not templates; *they must be typesafe given any possible construction which satisfies the constraints*, not just under the set that they are actually constructed from in a particular program.

Next time I'll consider what impact these design decisions have on non-virtual instance methods called from within generic types.

</div>

</div>

</div>

