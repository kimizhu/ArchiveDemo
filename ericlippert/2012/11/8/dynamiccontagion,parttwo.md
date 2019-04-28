# Dynamic contagion, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/8/2012 6:43:00 AM

-----

[Last time I discussed how "dynamic" tends to spread through a program like a virus](http://blogs.msdn.com/b/ericlippert/archive/2012/11/05/dynamic-contagion-part-one.aspx): if an expression of dynamic type "touches" another expression then that other expression often also becomes of dynamic type. Today I want to describe one of the least well understood aspects of method type inference, which also uses a contagion model when "dynamic" gets involved.

Long-time readers know that [method type inference is one of my favourite parts of the C\# language](http://blogs.msdn.com/b/ericlippert/archive/tags/type+inference/); for new readers who might not be familiar with the feature, let me briefly describe it. The idea is that when you have a method, say, Select\<A, R\>(IEnumerable\<A\> items, Func\<A, R\> projection), and a call to the method, say Select(customers, c=\>c.Name), then we infer that you meant to say Select\<Customer, string\>(customers, c=\>c.Name), rather than making you spell it out. In that case, we would first infer that the list of customers is an IEnumerable\<Customer\> and therefore the type argument corresponding to A is Customer. From that we would infer that lambda parameter c is of type Customer, and therefore the result of the lambda is string, and therefore type argument corresponding to R is string. This algorithm is already complicated, but when dynamic gets involved, it gets downright weird.

The problem that the language designers faced when deciding how method type inference works with dynamic is exacerbated by [our basic design goal for dynamic, that I mentioned two weeks ago](http://blogs.msdn.com/b/ericlippert/archive/2012/10/22/a-method-group-of-one.aspx): the runtime analysis of a dynamic expression honours all the information that we deduced at compile time. We only use the deduced-at-runtime types for the parts of the expression that were actually dynamic; the parts that were statically typed at compile time remain statically typed at runtime, not dynamically typed. Above we inferred R after we knew A, but what if "customers" had been of type dynamic? We now have a problem: depending on the runtime type of customers, type inference might succeed dynamically even though it seems like it must fail statically. But if type inference fails statically then the method is not a candidate, and, as we discussed two weeks ago, if the candidate set of a dynamically-dispatched method group is empty then overload resolution fails at compile-time, not at runtime. So it seems that type inference must succeed statically\!

What a mess. How do we get out of this predicament? The spec is surprisingly short on details; it says only:

> Any type argument that does not depend directly or indirectly on an argument of type dynamic is inferred using \[the usual static analysis rules\]. The remaining type arguments are *unknown.* \[...\] Applicability is checked according to \[the usual static analysis rules\] ignoring parameters whose types are *unknown.* (\*)

So what we have here is essentially another type that spreads via a contagion model, the "unknown" type. Just as "possibly infected" is the transitive closure of the exposure relation in simplistic epidemiology, "unknown" is the transitive closure of the "depends on" relation in method type inference.

For example, if we have:

void M\<T, U\>(T t, L\<U\> items)

with a call

M(123, dyn);

Then type inference infers that T is int from the first argument. Because the second argument is of dynamic type, and the formal parameter type involves type parameter U, we "taint" U with the "unknown type".

When a tainted type parameter is "fixed" to its final type argument, we ignore all other bounds that we have computed so far, even if some of the bounds are contradictory, and infer it to be "unknown". So in this case, type inference would succeed and we would add M\<int, unknown\> to the candidate set. As noted above, we skip applicability checking for arguments that correspond to parameters whose types are in any way tainted.

But where does the transitive closure of the dependency relationship come into it? In the C\# 4 and 5 compilers we did not handle this particularly well, but in Roslyn we now actually cause the taint to spread. Suppose we have:

void M\<T, U, V\>(T t, L\<U\> items, Func\<T, U, V\> func)

and a call

M(123, dyn, (t, u)=\>u.Whatever(t));

We infer T to be int and U to be unknown. We then say that V depends on T and U, and so infer V to be unknown as well. Therefore type inference succeeds with an inference of M\<int, unknown, unknown\>.

The alert reader will at this point be protesting that no matter what happens with method type inference, this is going to turn into a dynamic call, and that lambdas are not legal in dynamic calls in the first place. However, we want to get as much high-quality analysis done as possible so that IntelliSense and other code analysis works correctly even in badly broken code. It is better to allow U to infect V with the unknown taint and have type inference succeed, as the specification indicates, than to bail out early and have type inference fail. And besides, if by some miracle we do in the future allow lambdas to be in dynamic calls, we'll already have a sensible implementation of method type inference.

**Next time** on Fabulous Adventures in Coding: **Fabulous Adventures**\!

-----

(\*) That last clause is a bit unclear in two ways. First, it really should say "whose types are *in any way* unknown". L\<unknown\> is considered to be an unknown type. Second, along with skipping applicability checking we also skip constraint satisfaction checking. That is, we assume that the runtime construction of L\<unknown\> will provide a type argument that satisfies all the necessary generic type constraints.

