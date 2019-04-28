# How do we ensure that method type inference terminates?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/2/2012 9:48:31 AM

-----

I missed the party. I was all set to be on that [massive](http://blogs.msdn.com/b/somasegar/archive/2012/10/01/typescript-javascript-development-at-application-scale.aspx) wave of [announcements](https://channel9.msdn.com/posts/Anders-Hejlsberg-Introducing-TypeScript) about [TypeScript](http://www.typescriptlang.org/), and then a family emergency kept me away from computers from Thursday of last week until just now, and I did not get my article in the queue. Suffice to say that I am **SUPER EXCITED** about TypeScript. Long-time readers of this blog know that I [have a long history with ECMAScript](http://blogs.msdn.com/b/ericlippert/archive/tags/jscript/), and I've wanted features like this for quite some time. Since I've missed the first wave I'm going to digest some of the initial reactions and hopefully post something a bit more thoughtful farther down the road.

Therefore, rather than talking about TypeScript today, here's a question I got from a coworker recently: since it is obviously important that the C\# compiler not go into infinite loops, **how do we ensure that the method type inference algorithm terminates?**

The answer is quite straightforward actually, but if you are not familiar with method type inference then this article is going to make no sense. Check out my [type inference archive](http://blogs.msdn.com/b/ericlippert/archive/tags/type+inference/), and specifically [this video](http://blogs.msdn.com/b/ericlippert/archive/2006/11/17/a-face-made-for-email-part-three.aspx), if you need a refresher.

Method type inference since C\# 3.0 basically works like this: we create a set of **bounds** on each method type parameter. We then "fix" each type parameter to a member of its bounds set. Once every type parameter is fixed, method type inference has succeeded. If any type parameter cannot be fixed for some reason then type inference fails. We ensure that type inference terminates by going into a loop. If we manage to make it through the body of the loop without fixing at least one type parameter then type inference fails. Therefore, the type inference loop can run at most n times if the method has n type parameters.

That's a bit highfalutin; let me flesh that out a bit. A "bound" is nothing more than a type, and a bound can be "upper", "lower" or "exact". For example, suppose we have a type parameter T with three bounds: a lower bound of Giraffe, an exact bound of Mammal, and an upper bound of Animal. Let's say that Animal is a "[larger](http://blogs.msdn.com/b/ericlippert/archive/2007/10/16/covariance-and-contravariance-in-c-part-one.aspx)" type than Mammal (because all Mammals are Animals but not all Animals are Mammals, thus Animal must be the larger type), and Giraffe is a "smaller" type than Mammal. Given this set of bounds we know that T must be inferred to be first, either Giraffe or a type larger than Giraffe, because Giraffe is a lower bound; you can't infer a type smaller than Giraffe. Second, we know that T must be Mammal, exactly. And third, we know that T must be either Animal or a type smaller than Animal, because Animal is an upper bound. We cannot infer a type larger than Animal. The C\# compiler deduces that Mammal is the only type in the set that meets all three requirements, and so T would be fixed to Mammal. If there are multiple types in the set that meet all the requirements (which of course cannot happen if there are any exact bounds\!) then we pick the largest such type. (\*)

The interesting part of method type inference is how we deal with lambdas. Suppose we have a method Select\<A, R\>(I\<A\>, Func\<A, R\>) where the second argument is c=\>c.Name. We say that A is an "input" type parameter and R is an "output" type parameter. (It is of course possible for a type parameter to be both an input and output type parameter\!) Furthermore, we say that R "depends on" A, because the type of A could possibly determine the type of R. (Of course the "depends" relationship can be cyclic.)

The type inference algorithm, at a high level, goes like this:

  - Add bounds to type parameters based on all non-lambda arguments, and all lambda arguments where the delegate type has no type parameters in its inputs.
  - Loop
      - Is every type parameter fixed?
          - Type inference has succeeded. Terminate the algorithm.
      - Is there any lambda argument converted to a delegate type where the inputs of the delegate type are all known and the output type involves an unfixed type parameter?
          - Deduce the return type of all such lambdas and make inferences that add bounds to the corresponding delegate's output types.
      - Is there any unfixed, bounded type parameter that does not appear in an output type of a delegate that has unfixed input types?
          - Fix all such type parameters and go back to the top of the loop.
      - Is there any unfixed, bounded type parameter such that an unfixed type parameter depends on it, directly or indirectly?
          - Fix all such type parameters and go back to the top of the loop.
      - If we make it here then we failed to make progress; we have just as many fixed type parameters as we started with. Type inference fails. Terminate the algorithm.

So, for example, if we had Select(customers, c=\>c.Name); where customers implements I\<Customer\> then we start by inferring that A has a lower bound of Customer (\*\*). We have no lambda arguments that correspond to formal parameters where the delegate type has no type parameters in its inputs, so we enter the loop.

Is every type parameter fixed? No.

Is there any lambda argument converted to a delegate type where the inputs are known and the output involves an unfixed type parameter?

No. There is a lambda argument converted to a delegate type, and the output involves unfixed type parameter R, but the input type is A and A is not fixed. So we have no inferences to make.

Is there an unfixed type parameter that has bounds and does not appear in an output type of a delegate that has unfixed input types?

Yes. A has bounds and does not appear as an output type, period.

Therefore we fix A. It has only one bound, Customer, so we fix it to Customer. We have made progress, so we go back to the top of the loop.

Is every type parameter fixed? No.

Is there any lambda argument converted to a delegate type where the inputs are known and the output involves an unfixed type parameter? Yes\!

So now we make an inference. A is fixed to Customer, so we add the type of Customer.Name, say, string, as a lower bound to R.

Now we must fix something. Is there an unfixed type parameter that has bounds and does not appear in an output type of a delegate that has unfixed input types?

Yes. R is unfixed, it has bounds, and it appears as an output type of a delegate that has fixed input types, so it is a candidate for fixing. We fix R to its only bound, string, and start the loop again.

Is every type parameter fixed? Yes. So we're done.

This technique of preventing infinite loops by requiring that each loop iteration make progress is quite useful, and clearly in this case it guarantees that the algorithm executes the loop no more times than there are type parameters to fix.

You might wonder if it is therefore the case that method type inference is O(n) in the number of type parameters. It turns out that it is not, for several reasons. First, as a practical matter it only makes sense to determine the asymptotic order of an algorithm if the size of the problem is likely to become large. I've never seen a method with more than five type parameters in the wild, and even that is pretty unusual. Most generic methods have one or two type parameters. Second, doing the analysis of the lambdas is the expensive part, and it only really makes sense to analyze the behaviour of the most expensive part. We [already know that analyzing lambdas is, worst case, an NP-HARD problem](http://blogs.msdn.com/b/ericlippert/archive/2007/03/28/lambda-expressions-vs-anonymous-methods-part-five.aspx) so whether or not method type inference is O(some polynomial) is possibly not that relevant. Third, you'll notice that in my sketch of the algorithm we have to answer questions like "is there any unfixed type parameter that has an unfixed type parameter that depends on it?" This requires solving a graph-traversal problem, whose asymptotic cost we have not analyzed\! I won't take you through the boring analysis, but suffice to say there could be O(n<sup>2</sup>) dependency relationships that each cost O(n) to analyze, and we could go through the loop n times, for an extremely unlikely worst case of O(n<sup>4</sup>). The implementation of this algorithm is actually O(n<sup>2</sup>) in the common case; because n is likely to be small, as I said, we have not put the effort into more sophisticated algorithms that can solve these graph problems even faster in the asymptotic case.

-----

(\*) Note that this algorithm is consistent with other type inference features in C\# in two ways. First, when asked to infer a best type from a set of types, we always choose a type from the set. We never say "well we have Dog and Cat in the set so let's infer Mammal". Second, when faced with multiple possible "best" types, we pick the largest. There is an argument to be made for picking the smallest, but picking the largest seems to match more people's intuitions of what the right choice is.

(\*\*) Assuming that the type I\<T\> is covariant in T. If it were contravariant then we would deduce an upper bound, and if it were invariant then we would deduce an exact bound. [See my series on variance if that is not clear.](http://blogs.msdn.com/b/ericlippert/archive/tags/covariance+and+contravariance/)

