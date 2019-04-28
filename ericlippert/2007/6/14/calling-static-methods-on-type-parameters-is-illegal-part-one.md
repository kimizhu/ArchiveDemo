<div id="page">

# Calling static methods on type parameters is illegal, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/14/2007 4:24:00 PM

-----

<div id="content">

<div class="mine">

A developer passed along a question from a customer to our internal mailing list for C\# questions the other day. Consider the following:

<span class="code"> </span>

public class C { public static void M() { /\*whatever\*/ } }  
public class D : C { public new static void M() { /\*whatever\*/ } }  
public class E\<T\> where T : C { public static void N() { T.M(); } }

That's illegal. We do not allow you to call a static method on a type parameter. The question is, why? We know that <span class="code">T</span> must be a <span class="code">C</span>, and <span class="code">C</span> has a static method <span class="code">M</span>, so why shouldn't this be legal?

I hope you agree that in a sensibly designed language *exactly one* of the following statements has to be true:

1\) This is illegal.  
2\) <span class="code">E\<T\>.N()</span> calls <span class="code">C.M()</span> no matter what <span class="code">T</span> is.  
3\) <span class="code">E\<C\>.N()</span> calls <span class="code">C.M()</span> but <span class="code">E\<D\>.N()</span> calls <span class="code">D.M()</span>.

(If there is a fourth possible sensible behaviour which I have missed, I am happy to consider its merits.)

If we pick (2) then this is both potentially misleading and totally pointless. The user will reasonably expect <span class="code">D.M()</span> to be called if <span class="code">T</span> is a <span class="code">D</span>. Why else would they go to the trouble of saying <span class="code">T.M()</span> instead of <span class="code">C.M()</span> if they mean "always call <span class="code">C.M()</span>"?

If we pick (3) then we have violated the core design principle of static methods, the principle that gives them their name. Static methods are called “static” because *it can always be determined exactly, at compile time, what method will be called*. That is, the method can be resolved solely by *static analysis* of the code.

That leaves (1).

Related questions come up frequently, in various forms. Usually people phrase it by asking me why C\# does not support “virtual static” methods. I am always at a loss to understand what they could possibly mean, since “virtual” and “static” are opposites\! “virtual” means “determine the method to be called based on run time type information”, and “static” means “determine the method to be called solely based on compile time static analysis”.

(Then again, people occasionally accuse me of “dogmatic skepticism”. Since “dogmatic” and “skeptical” are opposites, I am never sure quite what they mean either. My conclusion: people sometimes say strange things.)

Really what people want I think is yet another kind of method, which would be none of static, instance or virtual. We could come up with a kind of method which behaved like our option (3) above. That is, a method associated with a type (like a static), which does not take a non-nullable “this” argument (unlike an instance or virtual), but one where the method called would depend on the constructed type of <span class="code">T</span> (unlike a static, which must be determinable at compile time).

I’m not sure whether the CLR generic system supports codegenning such a beast, but other than that, I don’t see any in-principle reason why such a thing would be difficult to do. But we do not add language features just because we can; we add them when the compelling benefit outweighs the costs. No one has yet made the case to me that this kind of method would be really useful.

[Next time](http://blogs.msdn.com/b/ericlippert/archive/2007/06/18/calling-static-methods-on-type-parameters-is-illegal-part-two.aspx) I'll answer the follow-up question: why *isn't* this determined at compile time? That will take us into the subtle differences between *templates* and *generic types*. They are often confused.

</div>

</div>

</div>

