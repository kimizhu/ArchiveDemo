# Why Is The Return Type Parameter Last?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/25/2009 1:09:00 PM

-----

The generic delegate type Func\<A, R\> is defined as delegate R Func\<A, R\>(A arg). That is, the argument type is to the *left* of return type in the *declaration* of the generic type parameters, but to the *right* of the return type when they are *used*. What’s up with that? Wouldn’t it be a lot more natural to define it as delegate R Func\<R, A\>(A arg), so that the R’s and A’s go together?

Maybe in C\# it would, but in this case, it’s C\# that’s the crazy one.

When we speak it in English, the argument type comes before the return type. We say “length is function *from string to int*”, not “length is a function *to int from string*”.

When we write it in mathematical notation, we say that a function’s domain and range are defined as **f:A→R** – again, the “return type” comes last.

And in many languages, the return type of a function comes in the sensible position. In VB it’s Function F(arg As A) As R.  In JScript .NET it’s function F(arg : A) : R.

And finally, consider higher-order functions; say, a function from A to a function from B to C. You want to think of this as **A→(B→C)**; do you really want to write that as Func\<Func\<C, B\>, A\> ? This is completely backwards. Surely you want **A→(B→C)** to be represented as Func\<A, Func\<B, C\>\>.

C\# gets it wrong because C\# inherits the basic pieces of its syntax from C, and C gets it wrong. Well, no, rather, it would be more fair to say that C is a non-typesafe, non-memory-managed language where it is vitally important that the code maintainer understands the lifetime and type of the data in every variable. Given that unfortunate situation, it makes sense to emphasize the storage mechanism first, and then the semantics second. Therefore in C you put the storage metadata first (static int customerCount;) rather than the semantics first (it could have been var customerCount: static int;). Once you’re in the position where the type comes first on variable declarations, it makes sense to apply the same rule to all other kinds of declarations – methods, formal parameters, and so on.

It might have been nicer back in the early days of C\# to say “you know, we have a type-safe, memory-managed language, let’s do what VB does, de-emphasize the type mechanism and put the type as an annotation on the end”. We could then make that consistent throughout the language so that Func\<A, R\> referred to delegate Func\<A, R\>(arg : A ) : R. But that ship has sailed and we’re stuck with the declaration syntax we’ve got.

