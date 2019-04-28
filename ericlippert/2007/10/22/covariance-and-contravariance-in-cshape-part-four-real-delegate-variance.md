<div id="page">

# Covariance and Contravariance in C\#, Part Four: Real Delegate Variance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/22/2007 7:39:00 AM

-----

<div id="content">

<div class="mine">

In the last two posts I discussed the two kinds of variance that C\# already has -- [array covariance](http://blogs.msdn.com/ericlippert/archive/2007/10/17/covariance-and-contravariance-in-c-part-two-array-covariance.aspx) and [member-group-to-delegate conversion covariance](http://blogs.msdn.com/ericlippert/archive/2007/10/19/covariance-and-contravariance-in-c-part-three-member-group-conversion-variance.aspx) (on return types) and contravariance (on formal parameter types).

Today I want to generalize the latter kind of variance.

In C\# 3.0 today, even though it is legal to assign a *typeless* *method group* for a function that returns a <span class="code">Giraffe</span> to a variable of type <span class="code">Func\<Animal\></span>, it is not legal to assign a *typed expression* of type <span class="code">Func\<Giraffe\></span> to a <span class="code">Func\<Animal\></span>. **Generic delegate types are always invariant in C\# 3.0.** That seems weak.

Suppose we had the ability to declare type parameters of generic delegate types as being covariant or contravariant. For the sake of brevity (and consistency with existing notation in the CLR specification) I will notate a covariant type parameter with a <span class="code">+</span> and a contravariant type parameter with a <span class="code">-</span>.

This is not a particularly compelling notation; I will discuss its deficiencies in a later post. But for now we'll stick with it.The way to remember what it means is that a **plus** means "this type argument is allowed to get **bigger** upon assignment", and similarly for minus.

Consider for example our standard function:

<span class="code"> </span>

delegate R Func\<A, R\>(A a);

Since <span class="code">R</span> appears only in the returns and <span class="code">A</span> appears only in the formal parameter list, we can make <span class="code">R</span> covariant and <span class="code">A</span> contravariant(‡):

<span class="code"> </span>

delegate R Func\< -A, +R \>(A a);

So again, you can think of this as "you can make <span class="code">A</span> smaller or <span class="code">R</span> bigger" (or, of course, both). For example:

<span class="code"> </span>

Func\<Animal, Giraffe\> f1 = whatever;  
Func\<Mammal, Mammal\> f2 = f1;

Normally in C\# this assignment would be illegal because the delegates are parameterized by different types. But since we have made <span class="code">Func</span> variant in both its type parameters, *this assignment would become legal were we to add this kind of variance to a hypothetical future version of C\#.*

Does that make sense so far?

(‡) This rule of thumb is not always correct\! Sometimes the input parameters need to be of a *covariant* type parameter. I shall discuss just such a situation next time, and I promise that it will hurt your brain.

</div>

</div>

</div>

