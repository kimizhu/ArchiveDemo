# What's the difference between covariance and assignment compatibility?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/30/2009 6:51:00 AM

-----

I've written a lot about this already, but I think one particular point bears repeating.

As we're getting closer to shipping C\# 4.0, I'm seeing a lot of documents, blogs, and so on, attempting to explain what "covariant" means. This is a tricky word to define in a way that is actually meaningful to people who haven't already got degrees in category theory, but it can be done. And I think it's important to avoid defining a word to mean something other than its actual meaning.

A number of those documents have led with something like:

"Covariance is the ability to assign an expression of a more specific type to a variable of a less specific type. For example, consider a method M that returns a Giraffe. You can assign the result of M to a variable of type Animal, because Animal is a less specific type that is compatible with Giraffe. Methods in C\# are 'covariant' in their return types, which is why when you create a covariant interface, it is indicated with the keyword 'out' -- the returned value comes 'out' of the method."

But that's **not at all** what covariance means. That's describing "assignment compatibility" -- the ability to assign a value of a more specific type to a storage of a compatible, less specific type is called "assignment compatibility" because the two types are compatible for the purposes of verifying legality of assignments.

So what does covariance mean then?

First off, we need to work out precisely what the adjective "covariant" applies to. I'm going to get more formal for a bit here, but try to keep it understandable.

Let's start by not even considering types. Let's think about integers. (And here I am speaking of actual mathematical integers, not of the weird behaviour of 32-bit integers in unchecked contexts.) Specifically, we're going to think about the ≤ relation on integers, the "less than or equal to" relation. (Recall that of course a "relation" is a function which takes two things and returns a bool which indicates whether the given relationship holds or does not hold.)

Now let's think about a projection on integers. What is a projection? A projection is a function which takes a single integer and returns a new integer. So, for example, z → z + z is a projection; call it D for "double".  So are z → 0 - z, N for "negate" and z → z \* z, S for "square".

Now, here's an interesting question. Is it always the case that (x ≤ y) = (D(x) ≤ D(y))?  Yes, it is. If x is less than y, then twice x is less than twice y. If x is equal to y then twice x is equal to twice y. And if x is greater than y, then twice x is greater than twice y. The projection D preserves the direction of size.

What about N? Is it always the case that (x ≤ y) = (N(x) ≤ N(y))?  Clearly not. 1 ≤ 2 is true, but -1 ≤ -2 is false. But we notice that the reverse is always true\!  (x ≤ y) = (N(y) ≤ N(x)). The projection N reverses the direction of size.

What about S? Is it always the case that (x ≤ y) = (S(x) ≤ S(y))? No. -1 ≤ 0 is true, but S(-1) ≤ S(0) is false. What about the opposite? Is it always the case that (x ≤ y) = (S(y) ≤ S(x)) ? Again, no. 1 ≤ 2 is true, but S(2) ≤ S(1) is false. The projection S does not preserve the direction of size, and nor does it reverse it.

The projection D is "covariant" -- it preserves the ordering relationship on integers. The projection N is "contravariant". It reverses the ordering relationship on integers. The projection S does neither; it is "invariant".

Now I hope it is more clear exactly what is covariant or contravariant. The integers themselves are not variant, and the "less than" relationship is not variant. It's **the projection** that is covariant or contravariant -- the rule for taking an old integer and making a new one out of it.

So now lets abandon integers and think about reference types. Instead of the ≤ relation on integers, we have the ≤ relation on reference types. A reference type X is smaller than (or equal to) a reference type Y if a value of type X can be stored in a variable of type Y. That is, if X is "assignment compatible" with Y.

Now consider a projection from types to types. Say, the projection "T goes to IEnumerable\<T\>".  That is, we have a projection that takes a type, say, Giraffe, and gives you back a new type, IEnumerable\<Giraffe\>. Is that projection covariant in C\# 4?  Yes. It preserves the direction of ordering. A Giraffe may be assigned to a variable of type Animal, and therefore an sequence of Giraffes may be assigned to a variable that can hold a sequence of Animals.

We can think of generic types as "blueprints" that produce constructed types. Let's take the projection that takes a type T and produces IEnumerable\<T\> and simply call that projection "IEnumerable\<T\>". We can understand from context when we say "IEnumerable\<T\> is covariant" what we mean is "the projection which takes a reference type T and produces a reference type IEnumerable\<T\> is a covariant projection". And since IEnumerable\<T\> only has one type parameter, it is clear from the context that we mean that the parameter to the projection is T. After all, it is a lot shorter to say "IEnumerable\<T\> is covariant" than that other mouthful.

So now we can define covariance, contravariance and invariance. A generic type I\<T\> is **covariant** (in T) if construction with reference type arguments **preserves** the direction of assignment compatibility. It is **contravariant** (in T) if it **reverses** the direction of assignment compatibility. And it is **invariant** if it does neither. And by that, we simply are saying in a concise way that the **projection** which takes a T and produces I\<T\> is a covariant/contravariant/invariant projection.

 UPDATE: My close personal friend (and fellow computer geek) Jen notes that in the Twilight series of novels, the so-called "werewolves" (who are not transformed by the full moon and therefore not actually werewolves) maintain their rigid social ordering in both wolf and human form; the projection from human to wolf is covariant in the social-ordering relation. She also notes that in high school, programming language geeks are at the bottom of the social order, but the projection to adulthood catapults them to the top of the social order, and therefore, growing up is contravariant. I am somewhat skeptical of the latter claim; the former, I'll take your word for it. I suppose the question of how social order works amongst teenage werewolves who are computer geeks is a subject for additional research. Thanks Jen\!

