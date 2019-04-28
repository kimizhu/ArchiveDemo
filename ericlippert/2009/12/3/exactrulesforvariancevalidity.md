# Exact rules for variance validity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/3/2009 6:32:00 AM

-----

I thought it might be interesting for you all to get a precise description of how exactly it is that we determine when it is legal to put "in" and "out" on a type parameter declaration in C\# 4. I'm doing this here because (1) it's of general interest, and (2) our attempt to make a more human-readable version of this algorithm in the draft C\# 4.0 specification accidentally introduced some subtle errors. We're working on correcting those errors for the final release of the specification; until then, *these definitions are the definitions I actually worked from to do the implementation*, so they're accurate.

These definitions are pretty much stolen straight from the CLI spec section on variance; I am indebted to its authors for their careful and precise definitions.

The first things we need to define are three kinds of "validity" for **types**. I want to define "valid covariantly", "valid contravariantly" and "valid invariantly", but only as applied to **types**. We'll talk about what makes an interface **declaration** a valid one later; we need these definitions first.

Before we get into a precise definition, I want to talk about what "valid covariantly" means logically. The idea that we are attempting to capture here boils down to "the type we're talking about is *not contravariant*". Whether it is *covariant*, we don't really care. All we care about it is that **if it is valid covariantly, then it is not contravariant**. Similarly, by "valid contravariantly", we mean "not covariant".

This leads us to our first brain-hurting definition: A type is "**valid invariantly**" if it is **both** valid covariantly and valid contravariantly. That sounds a bit crazy -- how can it be valid invariantly if it is both covariant and contravariant? But "valid covariantly" does not mean "is covariant", it means "it's guaranteed to not be contravariant". So if a type is valid covariantly and valid contravariantly, then it is guaranteed to be neither contravariant nor covariant, and therefore must be invariant.

Anyway. We want to take any specific type and classify it as valid covariantly, valid contravariantly, or both, a.k.a., valid invariantly. (We don't need to worry about coming up with a term for the "or none of the above" case because in practice that case is never relevant.)

Nested types present a problem; we might have an interface I\<T\> inside a class C\<U\>. For simplicity's sake, let's assume that the type C\<U\>.I\<T\> is analyzed as if it were C.I\<U, T\>; logically that's the same thing and it will keep the notation simpler below. Similarly, we could have a "generic enum", C\<U\>.E; suspend disbelief, pretend that C.E\<U\> is legal and move on. Remember, for the purposes of working out problems in type algebra, where the generic type arguments go lexically is irrelevant; all that matters is their number and order.

Nullable types are considered to be the generic struct type Nullable\<T\>.

A type is **valid covariantly** if it is:

1\) a pointer type, or a non-generic class, struct, enum, delegate or interface type.

This should make sense. Remember, what we're getting at is "not contravariant". Pointers and non-generic types are by definition not generic and therefore not variant either. These are the easy cases.

2\) An array type T\[\] where T is valid covariantly.

Because we have array covariance in C\#, but not array contravariance, we can make arrays valid covariantly. Again, remember, "valid covariantly" means "not contravariant".

3\) A generic type parameter type, if it was not declared as being contravariant.

Generic type parameter types are of course types as far as the compiler is concerned. If you're inside the declaration of a generic interface then you can use its type parameters as types. In that context, such types are valid covariantly if they were not declared as contravariant (in).

4\) A constructed class, struct, enum, interface or delegate type X\<T1, ... Tk\> might be valid covariantly. To determine if it is, we examine each type argument differently, depending on whether the corresponding type parameter was declared as covariant (out), contravariant (in), or invariant (neither). (Of course the generic type parameters of classes and structs will never be declared 'out' or 'in'; they will always be invariant.) If the ith type parameter was declared as covariant, then Ti must be valid covariantly. If it was declared as contravariant, then Ti must be valid contravariantly. If it was declared as invariant, then Ti must be valid invariantly.

As one would expect, covariant validity preserves the direction of validity; covariant parameters must be valid covariantly, contravariant parameters must be valid contravariantly.

OK, I hope that was relatively painless. The rules for contravariant validity are similar; as one would expect from the "backwards" nature of contravariance, the directions are reversed in the complicated case:

A type is **valid contravariantly** if it is:

1\) a pointer type or non-generic class, struct, enum, delegate or interface type.

2\) An array type T\[\] where T is valid contravariantly.

Arrays are covariant in their element type. Covariance preserves the direction of variance. Therefore, to be valid contravariantly, an array (covariant in its element type) must be contravariantly valid in its element type.

3\) A generic type parameter type, if it was not declared as being covariant.

Remember, "valid contravariantly" means "not covariant". It does not mean "contravariant".

4\) A constructed class, struct, enum, interface or delegate type X\<T1, ... Tk\> might be valid contravariantly. If the ith type parameter was declared as contravariant, then Ti must be valid covariantly. If it was declared as covariant, then Ti must be valid contravariantly. If it was declared as invariant, then Ti must be valid invariantly. (Again, of course all the type parameters contributed from classes and structs will be declared invariant.)

As one might expect, contravariant validity *reverses* the direction of validity.

Now perhaps you see why we wanted to rewrite these definitions into something more human-readable for the spec. And perhaps you also see why we accidentally introduced errors in doing so; bending your brain around all this logic is not always easy.

OK, now that we've got that, we can make a definition of what it means for an interface to be valid. An interface must meet the following conditions:

\* The return types of all non-void interface methods must be valid covariantly.

\* Every formal parameter type of all interface methods must be valid contravariantly. (Invariantly if it is an out or ref parameter.)

\* For all generic methods on an interface, every constraint on the generic method type parameters must be valid contravariantly.

\* All its base interface types must be valid covariantly.

\* The type of a property or indexer must be valid covariantly if it has a "getter" and valid contravariantly if it has a "setter".

\* Any formal parameter types of an indexer must be valid contravariantly.

\* The delegate types of all its events must be valid contravariantly.

The first two are pretty straightforward; return types "go out" so they have to be valid covariantly, formal parameter types "go in", so they have to be valid contravariantly. But what's up with the third one? What do constraints on generic method type parameters have to do with interface validity?

Well, let's suspend the third rule and see what goes wrong.

 

interface I\<out T\>  
{  
    void M\<U\>() where U : T;  
    // illegal; this has to be valid contravariantly but it is a covariant type parameter constraint.  
    // Let it ride for now and demonstrate an error.  
}

class C\<T\> : I\<T\> { public void M\<U\>() {} }  
// the constraint is inherited implicitly and not re-stated.

I\<Giraffe\> igiraffe = new C\<Giraffe\>(); // C\<T\> implements I\<T\>  
I\<Animal\> ianimal = igiraffe; // interface is covariant in T  
ianimal.M\<Turtle\>(); // satisifies the constraint that U must be an Animal.

Uh oh. ianimal is really an instance of C\<Giraffe\>. The M\<U\> method on C\<Giraffe\> inherits a requirement that U inherit from Giraffe. Turtle does not inherit from Giraffe. Therefore we've just violated the constraint on M\<U\>. The only places where we can catch this is in the declaration; every other step is perfectly legal. Therefore, a constraint cannot be covariant. But if we make it contravariant (or invariant) then it all works out. For example, let's make a contravariant type parameter constraint:

 

interface I2\<in T\> // contravariant this time  
{  
    void M\<U\>() where U : T;  
}

class C2\<T\> : I2\<T\> { public void M\<U\>() {} }  
I2\<Animal\> i2animal = new C2\<Animal\>(); // C2\<T\> implements I2\<T\>  
I2\<Mammal\> i2mammal = i2animal; // interface is contravariant in T  
i2mammal.M\<Giraffe\>(); // satisifies the constraint that U must be an Animal.

And now everything is fine; the compile-time constraint checker verifies that Giraffe is Mammal; at runtime it must be Animal, and so the compiler has verified that by verifying that it is Mammal. 

The rules for **delegate declarations** are a straightforward simplification of the rules for interface declarations. To be a valid delegate declaration, the return type must be valid covariantly (or void), the formal parameter types must be valid contravariantly (or invariantly if they are out/ref), and any type parameter constraints must be valid contravariantly.

