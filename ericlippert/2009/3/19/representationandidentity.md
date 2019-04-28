# Representation and Identity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/19/2009 9:46:00 AM

-----

(Note: not to be confused with [Inheritance and Representation](http://blogs.msdn.com/b/ericlippert/archive/2011/09/19/inheritance-and-representation.aspx).)

I get a fair number of questions about the C\# cast operator. The most frequent question I get is:

short sss = 123;  
object ooo = sss;            // Box the short.  
int iii = (int) sss;         // Perfectly legal.  
int jjj = (int) (short) ooo; // Perfectly legal  
int kkk = (int) ooo;         // Invalid cast exception?\! Why?

Why? Because a boxed T can only be unboxed to T. (\*) Once it is unboxed, it’s just a value that can be cast as usual, so the double cast works just fine.

Many people find this restriction grating; they expect to be able to cast a boxed thing to *anything* that the unboxed thing could have been cast to. There are ways to do that, as we’ll see, but there are good reasons why the cast operator does what it does.

To understand why this design works this way it’s necessary to first wrap your head around the contradiction that is the cast operator. There are two (¤) basic usages of the cast operator in C\#:

  - My code has an expression of type B, but I happen to have more information than the compiler does. I claim to know for certain that at runtime, **this object of type B will actually always be of derived type D**. I will inform the compiler of this claim by inserting a cast to D on the expression. Since the compiler probably cannot verify my claim, the compiler might ensure its veracity by inserting a run-time check at the point where I make the claim. If my claim turns out to be inaccurate, the CLR will throw an exception.
  - I have an expression of some type T which **I know for certain is not of type U**. However, I have a well-known way of associating some or all values of T with an “equivalent” value of U. I will instruct the compiler to generate code that implements this operation by inserting a cast to U. (And if at runtime there turns out to be no equivalent value of U for the particular T I’ve got, again we throw an exception.)

The attentive reader will have noticed that these are *opposites*. A neat trick, to have an operator which means two contradictory things, don’t you think?

This dichotomy motivates yet another classification scheme for conversions (†).  We can divide conversions into **representation-preserving conversions** (B to D) and **representation-changing conversions** (T to U). (‡) We can think of representation-preserving conversions on reference types as those conversions which *preserve the identity of the object*. When you cast a B to a D, you’re not doing anything to the existing object; you’re merely *verifying* that it is actually the type you say it is, and moving on. The identity of the object and the bits which represent the reference stay the same. But when you cast an int to a double, the resulting bits are very different.

All the built-in reference conversions are identity-preserving (£). Obviously trivial “conversions” such as converting from int to int are also representation-preserving conversions. All user-defined conversions (§) and non-trivial value type conversions (such as converting from int to double) are representation-changing conversions. Boxing and unboxing conversions are all representation-changing conversions.

The representation-preserving conversions that are known to never fail often result in no codegen at all (₪). If a representation-preserving conversion could fail then a castclass instruction is emitted, which does a runtime check and throws if the check fails.

But each representation-changing conversion is handled in its own special way. User-defined conversions are resolved using a special version of the overload resolution algorithm, and generated as a call to the appropriate static method. Boxing and unboxing conversions are generated as box and unbox instructions. All the other built-in conversions (int to double, and so on) are generated as **custom sequences of instructions** that do the right conversion.

So now that you know that, consider what the compiler would have to do to make this work the way some people expect:

int kkk = (int) ooo;

All that the compiler knows is that ooo is of type object. It could be *anything*. Suppose it is a boxed int – then the compiler should generate an unboxing instruction. Suppose it is a boxed short. Then the compiler should unbox the short and then generate the custom sequence of instructions that convert a short to an int. Suppose it is a boxed double – same thing, but different instructions. And so on, for all the built-in conversions that go to integer.

This would be a huge amount of code to generate, and it would be very slow. The code is of course so large that you would want to put it in its own method and just generate a call to it. Rather than do that by default, and always generate code that is slow, large and fragile, instead we’ve decided that unboxing can only unbox to the exact type. If you want to call the slow method that does all that goo, it’s available – you can always call Convert.ToInt32, which does all that analysis at runtime for you. We give you the choice between “fast and precise” or “slow and lax”, and *the sensible default is the former*. If you want the latter then call the method.

That’s just the built-in conversions. Let’s continue imagining what would have to happen if we wanted *all possible conversions* to int to just work out correctly at runtime, instead of just bailing out early if the boxed thing is not an int.

Suppose the object is a Foo where there is a user-defined conversion from Foo (or one of its base classes) to int (or a type that int is explicitly convertible from, like, say, Nullable\<int\>). Then the compiler would need to generate a call to that conversion method, just as it would if the type had been known at compile time, and then possibly also generate the conversion from the return type of the method to int.

Remember, there could be arbitrarily many such conversion methods on arbitrarily many types. The type Foo and its conversion method might not even be defined in the assembly currently being compiled or any assembly referenced. Therefore the compiler would have to generate code to interrogate Foo at runtime, do the overload resolution analysis, and then dynamically spit the code to do the call.

Which is exactly what the compiler does in C\# 4.0 if the argument to the cast is of type “dynamic” instead of object. *The compiler actually generates code which starts a mini version of the compiler up again at runtime, does all that analysis, and spits fresh code*. This is not *fast*, but it is *accurate*, if that’s what you really need. (And the spit code is then cached so that the next time this call site is hit, it is much faster.)

I don’t think people *really* expect the compiler to start up again at runtime every time they cast an object to int; I think they just haven’t thought through carefully exactly how much analysis solving the problem would take. Rather a lot, it turns out.

\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) Or Nullable\<T\>.

(¤) There are others that are not germane to this discussion. For example, a third usage is “Everyone knows that this D is also of base type B; I want the compiler to treat this expression of type D as a B for overload resolution purposes.” That would clearly be an identity-preserving conversion.

(†) There are many ways to classify conversions; we already divide conversions into implicit/explicit, built-in/user-defined, and so on. For the purposes of this discussion we’ll gloss over the details of those other classifications.

(‡) I’m glossing over here that certain conversions that the C\# compiler thinks of as representation-changing are actually seen by the CLR verifier as representation-preserving. For example, the conversion from int to uint is seen by the CLR as representation-preserving because the 32 bits of a signed integer can be reinterpreted as an unsigned integer without changing the bits. These cases can be subtle and complex, and often have an impact on covariance-related issues; see next footnote.

I’m also ignoring conversions involving generic type parameters which are not known at compile time to be reference or value types. There are special rules for classifying those which would be major digressions to get into.

(£) This is why covariant and contravariant conversions of interface and delegate types require that all varying type arguments be of reference types. **To ensure that a variant reference conversion is always identity-preserving, all of the conversions involving type arguments must also be identity-preserving.** The easiest way to ensure that all the non-trivial conversions on type arguments are identity-preserving is to restrict them to be reference conversions.

(§) The rules of C\# prohibit all user-defined conversions that could possibly be identity-preserving coercions. More generally, all user-defined conversions that could possibly be any "standard" conversion are illegal.

(₪) Again, I’m ignoring irksome generic issues here. There are situations where humans can prove mathematically that two generic type parameters must be identical at runtime, but the verifier is not smart enough to make those same deductions and requires the compiler to emit type checks.

