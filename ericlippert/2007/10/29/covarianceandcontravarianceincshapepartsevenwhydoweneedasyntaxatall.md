# Covariance and Contravariance in C\# Part Seven: Why Do We Need A Syntax At All?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/29/2007 10:32:00 AM

-----

Suppose we were to implement generic interface and delegate variance in a hypothetical future version of C\#. What, hypothetically, would the syntax look like? There are a bunch of options that we could hypothetically consider.

Before I get into options though, let’s be bold. What about “no syntax at all”? That is why not just *infer* *variance on behalf of the user* such that everything magically just works?

Unfortunately this doesn’t fly, for several reasons.

First, it seems to me that variance ought to be something that you *deliberately* design into your interface or delegate. Making it just start happening with no control by the user works against that goal, and also can introduce breaking changes. (More on those in a later post\!)

Doing so automagically also means that as the development process goes on and methods are added to interfaces, the variance of the interface may change unexpectedly. This could introduce unexpected and far-reaching changes elsewhere in the program.

Second, attempting to do so introduces a new kind of cycle to the language analysis. We already have to detect things like cycles in base classes, cycles in base interfaces and cycles in generic type constraints, so this is in theory nothing new. But in practice, there are some issues.

In my previous post I did not discuss what additional restrictions we’d need to put on variant interfaces; one important restriction is that **a variant interface which inherits from another variant interface must do so in a manner which does not introduce problems in the type system**. Basically, all the rules for when a type parameter can be covariant or contravariant need to "flow through" to base interfaces. (This is vague, I know. I have a precise formal definition of what all the rules are which I may post at a later date. The exact rules are not important for the purposes of this discussion.)

For example, suppose the compiler was trying to deduce variance in this program:

 

interface IFrob\<T\> : IBlah\<T\> { }  
interface IBlah\<U\> {  
  IFrob\<U\> Frob();  
}

We might ask ourselves “is it legal for T to be variant in IFrob\<T\>?” To answer that question, we need to determine whether it is legal for U to be variant in IBlah. To answer that question we need to know whether it is legal for U to be variant in output type IFrob\<U\>, and hey, we’re back where we started\!

I would rather the compiler not go into an infinite loop when given this program. But clearly this is a perfectly *legal* program. When we detect a cycle in base classes, we can throw up our hands and say "you've got an illegal program". We cannot do that here. That complicates matters.

Third, even if we could figure out a way to solve the cycle problem, we would still have a problem with the case above. Namely, there are three possible logically consistent answers: “both invariant”, “+T, +U” and “-T, -U” all produce programs which would be typesafe. How would we choose?

We could get into even worse situations:

 

interface IRezrov\<V, W\> {  
  IRezrov\<V, W\> Rezrov(IRezrov\<W, V\> x);  
}

In this crazy interface we can deduce that “both invariant”, “ \<+V, -W\> and \<-V, +W\> are all possibilities. Again, how to choose?

And fourth, even if we could solve all those problems, I suspect that the performance of such an algorithm would be potentially very bad. This has “exponential growth” written all over it. We have other exponential algorithms in the compiler, but I'd rather not add any more if we can avoid it.

Thus, if we do add interface and delegate variance in some hypothetical future version of C\#, we will provide a syntax for it. Next time, some ideas on what that syntax could look like. (If you have bright ideas yourself, feel free to post them in comments\!)

