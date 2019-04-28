# Why do ref and out parameters not allow type variation?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/21/2009 9:36:00 AM

-----

Here's a [good question from StackOverflow](http://stackoverflow.com/questions/1207144/c-why-doesnt-ref-and-out-support-polymorphism/1207302#1207302):

If you have a method that takes an "X" then you have to pass an expression of type X **or something convertible to X**. Say, an expression of a type derived from X. But if you have a method that takes a "ref X", you have to pass a ref to a variable of type X, period. Why is that? Why not allow the type to vary, as we do with non-ref calls?

Let's suppose you have classes Animal, Mammal, Reptile, Giraffe, Turtle and Tiger, with the obvious subclassing relationships.

Now suppose you have a method void M(ref Mammal m). M can both read and write m. Can you pass a variable of type Animal to M? No. That would not be safe. That variable could contain a Turtle, but M will assume that it contains only Mammals. A Turtle is not a Mammal.

**Conclusion 1**: Ref parameters cannot be made "bigger". (There are more animals than mammals, so the variable is getting "bigger" because it can contain more things.)

Can you pass a variable of type Giraffe to M? No. M can write to m, and M might want to write a Tiger into m. Now you've put a Tiger into a variable which is actually of type Giraffe.

**Conclusion 2**: Ref parameters cannot be made "smaller".

Now consider N(out Mammal n).

Can you pass a variable of type Giraffe to N? No. As with our previous example, N can write to n, and N might want to write a Tiger.

**Conclusion 3**: Out parameters cannot be made "smaller".

Can you pass a variable of type Animal to N?

Hmm.

Well, why not? N cannot read from n, it can only write to it, right? You write a Tiger to a variable of type Animal and you're all set, right?

Wrong. The rule is not "N can only write to n". The rules are, briefly:

1\) N has to write to n before N returns normally. (If N throws, all bets are off.)  
2\) N has to write something to n *before it reads something from n*.

That permits this sequence of events:

  - Declare a field x of type Animal.
  - Pass x as an out parameter to N.
  - N writes a Tiger into n, which is an alias for x.
  - On another thread, someone writes a Turtle into x.
  - N attempts to read the contents of n, and discovers a Turtle in what it thinks is a variable of type Mammal.

That scenario -- using multithreading to write into a variable that has been aliased -- is awful and you should never do it, but it is possible.

UPDATE: Commenter Pavel Minaev correctly notes that there is no need for multithreading to cause mayhem. We could replace that fourth step with

N makes a call to a method which directly or indirectly causes some code to write a Turtle into x.

Regardless of how the variable's contents might get altered, clearly we want to make the type system violation illegal.

**Conclusion 4**: Out parameters cannot be made "larger".

There is another argument which supports this conclusion: "out" and "ref" are actually exactly the same behind the scenes. The CLR only supports "ref"; "out" is just "ref" where the compiler enforces slightly different rules regarding when the variable in question is known to have been definitely assigned. That's why it is illegal to make method overloads that differ solely in out/ref-ness; the CLR cannot tell them apart\! Therefore the rules for type safety for out have to be the same as for ref.

**Final conclusion**: Neither ref nor out parameters may vary in type at the call site. To do otherwise is to break verifiable type safety.

