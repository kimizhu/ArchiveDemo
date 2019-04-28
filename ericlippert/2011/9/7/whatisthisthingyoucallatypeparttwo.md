# What is this thing you call a "type"? Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/7/2011 10:58:00 AM

-----

Well that was entirely predictable; as I said [last time](http://blogs.msdn.com/b/ericlippert/archive/2011/08/29/what-is-this-thing-you-call-a-quot-type-quot-part-one.aspx), if you ask ten developers for a definition of "type", you get ten different answers. The comments to the previous article make for fascinating reading\!

Here's my attempt at describing what "type" means to me as a compiler writer. I want to start by considering just the question of what a type *is* and not confuse that with *how it is used*.

Fundamentally, **a type in C\# is a mathematical entity that obeys certain algebraic rules**, just as natural numbers, complex numbers, quaternions, matrices, sets, sequences, and so on, are mathematical entities that obey certain algebraic rules.

By way of analogy, I want to digress for a moment and ask the question "*what is a natural number?*" That is, what fundamentally characterizes the numbers 0, 1, 2, 3, ... that we use to describe the positions of items in a sequence and the sizes of everyday collections of things?

This question has received a lot of attention over the centuries; the definitive answer that we still use today was created by [Giuseppe Peano](http://en.wikipedia.org/wiki/Peano) in the 19th century. Peano said that a natural number is defined as follows:

  - Zero is a natural number.
  - Every natural number has a "successor" natural number associated with it.
  - No natural number has zero as its successor.
  - Unequal natural numbers always have unequal successors.
  - If you start from zero, take its successor, and then take the successor of that, and so on, you will eventually encounter every natural number. (\*)

Surprisingly, that's all you need. Any mathematical entity that satisfies those postulates is usable as a natural number. (\*\*) Notice that there's nothing in there whatsoever about adding natural numbers together, or subtracting, multiplying, dividing, taking exponents, and so on. All of those things can be bolted on as necessary. For example, we can say that addition is defined as follows:

  - (n + 0) = n
  - (n + (successor of m)) = ((successor of n) + m)

And you're done; you've got a recursive definition of addition. We can similarly define "less than":

  - (n \< 0) = false
  - (0 \< (successor of m)) = true
  - ((successor of n) \< (successor of m)) = (n \< m)

We can define every operation on natural numbers in this way; try defining multiplication, just for fun. (Hint: assume that you've already defined addition.)

We can come up with a similar "axiomatic" definition of "type":

  - Object is a type
  - "Declared" types are types (note that int, uint, bool, string, and so on can be considered "declared" types for our purposes; the runtime "declares" them for you.)
  - If T is a type and n is a positive integer then "n-dimensional array of T" is also a type.

And that's pretty much it as far as "safe" C\# 1.0 is concerned. (To be as strict as the Peano axioms for natural numbers we'd want to also throw in some similar safeguards; for example, we don't ever want to be in a situation where the type "one-dimensional array of double" has *type* *equality* with the type "int", just as we don't ever want to be in a situation where the successor of a natural number is zero.)

Things get a bit more complex when we throw in generic types and pointer types, but I'm sure you can see that we could come up with a precise axiomatic description of generic types and pointer types with a little bit of work. That is, type parameter declarations are types, generic type declarations constructed with the right number of type arguments are types, and so on.

We can then start piling on algebraic operations. Just as we defined "less than" on numbers, we can define the "is a subtype of" relation on types.

  - T \<: object is true for any type T that is not object
  - object \<: T is false for any type T
  - if S \<: T and T \<: U then S \<: U
  - ... and so on...

Just as I do not care how numbers are "implemented" in order to manipulate them algebraically, I also do not care how types are "implemented" in order to manipulate them with the rules of type algebra. All I care about are the axioms of the system, and the rules that define the algebraic operators that I've made up for my own convenience.

So there you go; we have a definition of "type" that does not say anything whatsoever about "a set of values" or "a name". A **type is just an abstract mathematical entity**, a bit like a number, that obeys certain defining axioms, and therefore can be manipulated algebraically by making up rules for useful operators -- just like numbers can be manipulated abstractly according to algebraic rules that we make up for them.

Now that we have a sketch of an axiomatic definition of what a type is, what are we going to do with it?

Perhaps the most fundamental purposes of static type checking in a programming language like C\# are to associate a type with every relevant **storage location**, to associate a type with every (†) relevant **compile-time** **expression**, and to then ensure that it is impossible for a value associated with an **incompatible** type to be written to or read from any storage location. A compile-time **proof** of runtime type safety (**‡**): that's the goal.

The key word there is **proof**; now that we have developed an axiomatic definition of "type" **we can start constructing proofs based on these axioms**. The C\# specification defines:

  - what type is associated with every compile-time expression; of course, expressions whose types cannot be determined might be program errors
  - what type is associated with every storage location
  - what constitutes an acceptable assignment from an expression of given type (**‡**‡**** to a storage location of a given type

The task of the compiler writer is then to implement those three parts of the specification: associating a type with every expression, associating a type with every storage location, and then **constructing a proof** that the assignment is valid given the rules. If the compiler is able to construct a proof then the assignment is allowed. The tricky bit is this: *the specification typically just gives the rules of the system, not a detailed algorithm describing how to construct proofs using those rules*. If **any** proof exists then the compiler is **required** to find it. Conversely, if **no** proof exists, the compiler is required to deduce that too and produce a suitable compile-time type error.

Unfortunately, as it turns out, **that's impossible in general**. A system where it is possible for every expression to be classified as either "type safe" or "type unsafe" in a finite number of logical steps is called a "decidable" system. As Gödel famously proved, natural number arithmetic as axiomatized above is [undecidable](http://en.wikipedia.org/wiki/On_Formally_Undecidable_Propositions_in_Principia_Mathematica_and_Related_Systems_I); there are statements in formal arithmetic that can be **neither proved nor disproved** in a finite number of logical steps. [Assignment type checking is also in general undecidable](http://research.microsoft.com/en-us/um/people/akenn/generics/FOOL2007.pdf) in programming languages with both nominal subtyping (**‡**‡**‡******) and generic variance. [As I mentioned a while back](http://blogs.msdn.com/b/ericlippert/archive/2008/05/07/covariance-and-contravariance-part-twelve-to-infinity-but-not-beyond.aspx), it turns out that it would be possible to put additional restrictions on type declarations such that nominal subtyping would become decidable, but we have not ever done this in C\#. Rather, when faced with a situation that produces an infinitely long proof of type compatibility, the compiler just up and crashes with an "expression was too complex to analyze" error. I'm hoping to fix that in a hypothetical future version of the compiler, but it is not a high priority because these situations are so unrealistic.

Fortunately, situations where type analysis is impossible in general, or [extremely time consuming](http://blogs.msdn.com/b/ericlippert/archive/2007/03/28/lambda-expressions-vs-anonymous-methods-part-five.aspx), are rare in realistic C\# programs; rare enough that we can do a reasonable job of writing a fast compiler.

**Summing up:** A type is an abstract mathematical entity defined by some simple axioms. The C\# language has rules for associating types with compile-time expressions and storage locations, and rules for ensuring that the expression type is compatible with the storage type. The task of the compiler writer is to use those axioms and algebraic rules to construct a proof or disproof that every expression has a valid type and that every assignment obeys the assignment compatibility rules.

Though the axioms of what makes a type are pretty simple, the rules of associating types to expressions and determining assignment compatibility are exceedingly complex; that's why the word "type" appears 5000 times in the specification. To a large extent, the C\# language is *defined* by its type rules.

\----------------------

(\*) I am of course oversimplifying here; more formally, the real axiom formally states that **proof by induction works on natural numbers**. That is: if a property is true of zero, and the property being true for a number implies the truth for its successor, then the property holds for all numbers.

(\*\*) It is instructive to consider why the third, fourth and fifth postulates are necessary. Without the third postulate, there need only be one number: zero, which is its own successor\! Without the fourth postulate we could say that there are only two numbers: the successor of zero is one, the successor of one is one. Without the fifth postulate there could be \*two\* zeros: "red zero" and "blue zero". The successor of red zero is red one, the successor of blue zero is blue one, and so on. In each case, the system without the axiom satisfies the remaining four axioms, but in each case it seems very contrary to our intuition about what a "natural number" is.

(†) We fudge this in C\# of course; null literals, anonymous functions and method groups are technically classified as "expressions without any type". However, in a legal program they must occur in a context where the type of the expression can be inferred from its surrounding context.

(**‡**) The cast operator gives the lie to this, of course; [one meaning of the cast operator](http://blogs.msdn.com/b/ericlippert/archive/2009/03/19/representation-and-identity.aspx) is "there is no compile-time proof that this is type safe; I assert that it is safe, so generate code that checks my assertion at runtime." Also, [unsafe array covariance makes this goal unreachable](http://blogs.msdn.com/b/ericlippert/archive/2007/10/17/covariance-and-contravariance-in-c-part-two-array-covariance.aspx).

(**‡**‡****) Again, this is fudged in a few places. For example, it is not legal to assign an expression of type int to a variable of type short, *unless* the expression of type int is a compile-time constant known to fit into a short.

(**‡**‡**‡******) That is, a language where you state the base type of a newly declared type. Like "interface IFoo\<T\> : IBar\<T\>" uses nominal subtyping.

