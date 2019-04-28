# Covariance and Contravariance in C\#, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/16/2007 11:04:00 AM

-----

I have been wanting for a long time to do a series of articles about covariance and contravariance (which I will shorten to “variance” for the rest of this series.)

I’ll start by defining some terms, then describe what variance features C\# 2.0 and 3.0 already support today, and then discuss some ideas we are thinking about for hypothetical nonexistant future versions of C\#.

As always, keep in mind that we have not even shipped C\# 3.0 yet. **Any of my musings on possible future additions to the language should be treated as playful hypotheses, rather than announcements of a commitment to ship any product with any feature whatsoever.**

Today: what do we mean by “covariance” and “contravariance”?

The first thing to understand is that for any two types T and U, exactly one of the following statements is true:

  - T is bigger than U.
  - T is smaller than U.
  - T is equal to U.
  - T is not related to U.

For example, consider a type hierarchy consisting of Animal, Mammal, Reptile, Giraffe, Tiger, Snake and Turtle, with the obvious relationships. ( Mammal is a subclass of Animal, etc.) Mammal is a bigger type than Giraffe and smaller than Animal, and obviously equal to Mammal. But Mammal is neither bigger than, smaller than, nor equal to Reptile, it’s just different.

Why is this relevant? Suppose you have a variable, that is, a storage location. Storage locations in C\# all have a type associated with them. **At runtime you can store an object which is an instance of an equal or smaller type in that storage location.** That is, a variable of type Mammal can have an instance of Giraffe stored in it, but not a Turtle.

This idea of storing an object in a typed location is a specific example of a more general principle called the “substitution principle”. That is, in many contexts you can often substitute an instance of a “smaller” type for a “larger” type.

Now we can talk about variance. Consider an “operation” which manipulates types. If the results of the operation applied to any T and U always results in two types T’ and U’ with the *same* relationship as T and U, then the operation is said to be “covariant”. If the operation *reverses bigness and smallness* on its results but *keeps equality and unrelatedness the same* then the operation is said to be “contravariant”.

That’s totally highfalutin and probably not very clear. Next time we’ll look at how C\# 3 implements variance at present.

