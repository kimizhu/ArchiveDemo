# Covariance and Contravariance, Part Eleven: To infinity, but not beyond

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/7/2008 9:56:00 AM

-----

UPDATE: Andrew Kennedy, author of the paper I reference below, was good enough to point out some corrections and omissions, which I have addressed. Thanks Andrew\!

As I've discussed at length in this space, we are considering adding [covariance and contravariance](http://blogs.msdn.com/ericlippert/archive/tags/Covariance+and+Contravariance/default.aspx) on delegate and interface types parameterized with reference types to a hypothetical future version of C\#. (See my earlier articles in this series for an explanation of the proposed feature.)

Variance is highly useful in mainstream cases, but exposes a really irksome problem in certain bizarre corner cases. Here's just one example.

Consider a "normal" interface contravariant in its sole generic type parameter, and a "crazy" invariant interface which extends the normal interface in a strange way:

 

public interface IN\<in U\> {}  
public interface IC\<X\> : IN\<IN\<IC\<IC\<X\>\>\>\> {}

This is a bit weird, but certainly legal.

Before I go on, I want to digress and talk a bit about why this is legal. Most people when they first see such a beast immediately say "but an interface cannot inherit from itself, that's an illegal circularity in the inheritance chain\!"

First off, no, that is not correct. Nowhere does the C\# specification make this kind of inheritance illegal, and in fact, a weak form of it must be legal. You want to be able to say:

 

interface INumber\<X\> : IComparable\<INumber\<X\>\> { ... }

that is, you want to be able to express that one of the guarantees of the INumber\<X\> contract is that you can always compare one number with another one. Therefore, it must be legal to use a type's name in a type argument of a generic base type.

However, all is not rosy. This particularly gross kind of inheritance that I give as an example is in fact illegal in the CLR, even though it is not illegal in C\#. This means that it is possible to have the C\# compiler generate an interface type which then cannot be loaded by the CLR. This unfortunate mismatch is troubling, and I hope in a future version of C\# to make the type definition rules of C\# as strict or stricter than those of the CLR. Until then, if it hurts when you do that, don't do it.

Second, unfortunately, the C\# compiler presently has numerous bugs in its cycle detector such that sometimes things which kinda look like cycles but are in fact not cycles are flagged as cycle errors. This just makes it all the more difficult for people to understand what is a legal cycle and what isn't. For example, the compiler today will incorrectly report that this is an illegal base class cycle, even though it clearly is not:

 

public class November\<T\> {}  
public class Romeo : November\<Romeo.Sierra.Tango\> {  
   public class Sierra {  
       public class Tango {}  
    }  
}

I have devised a new (and I believe *correct*\!) cycle detection algorithm implementation, but unfortunately it will not make it into the service release of the C\# 3 compiler. It will have to wait for a hypothetical future release. I hope to address the problem of bringing the legal type checker into line with the CLR at the same time.

Anyway, back to the subject at hand: crazy variance. We have the interfaces defined as above, and then give the compiler a little puzzle to solve:

 

IC\<double\> bar = whatever;  
IN\<IC\<string\>\> foo = bar;  // Is this assignment legal?

I am about to get into a morass of impossible-to-read generic names, so to make it easier on all of us, I am going to from now on abbreviate IN\<IC\<string\>\> as NCS. IC\<double\> will be abbreviated as CD. You get the idea I'm sure.

Similarly, I will notate "is convertible to by implicit reference conversion" by a right-pointing arrow. So the question at hand is true or false: CD→NCS ?

Well, let’s see. Clearly CD does not go to NCS directly. But (the compiler reasons) maybe CD’s base type does.

CD’s base type is NNCCD. Does NNCCD→NCS? Well, N is contravariant in its parameter so therefore this boils down to the question, does CS→NCCD ?

Clearly not directly. But perhaps CS has a base type which goes to NCCD. The base type of CS is NNCCS. So now we have the question does NNCCS→NCCD ?

Well, N is contravariant in its parameter, so this boils down to the question does CCD→NCCS ?

Let’s pause and reflect a moment here.

The compiler has “reduced” the problem of determining the truth of CD→NCS to the problem of determining the truth of CCD→NCCS\! If we keep on “reducing” like this then we’ll get to CCCD→NCCCS, CCCCD→NCCCCS, and so on.

I have a prototype C\# compiler which implements variance – if you try this, it says “fatal error, an expression is too complex to compile”.

I considered implementing an algorithm that is smarter about determining convertibility; the paper I reference below has such an algorithm. (Fortunately, the C\# type system is weak enough that determining convertibility of complex types is NOT equivalent to the halting problem; we can find these bogus situations both in principle and in practice. Interestingly, there are type systems in which this problem is equivalent to the halting problem, and type systems for which the computability of convertibility is still an open question.) However, given that we have many other higher priorities, it’s easier to just let the compiler run out of stack space and have a fatal error. These are not realistic scenarios from which we must sensibly recover.

This is just a taste of some of the ways that the type system gets weird. To get a far more in-depth treatment of this subject, you should read this excellent [Microsoft Research paper](http://research.microsoft.com/~akenn/generics/FOOL2007.pdf).

