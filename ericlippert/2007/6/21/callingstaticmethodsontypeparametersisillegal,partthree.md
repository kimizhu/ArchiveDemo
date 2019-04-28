# Calling static methods on type parameters is illegal, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/21/2007 12:37:00 PM

-----

There were lots of good comments on my [previous](http://blogs.msdn.com/ericlippert/archive/2007/06/14/calling-static-methods-on-type-parameters-is-illegal-part-one.aspx) [entries](http://blogs.msdn.com/ericlippert/archive/2007/06/18/calling-static-methods-on-type-parameters-is-illegal-part-two.aspx) in this series. I want to address some of them, but first I want to wrap this up by considering how a small change to the scenario makes it plausible to choose a different option.  Consider now the **non-static, non-virtual** instance method:

 

public class C { public void M() { /\*whatever\*/ } }  
public class D : C { public new void M() { /\*whatever\*/ } }  
public class E\<T\> where T : C { public static void N(T t) { t.M(); } }

I hope you agree that in a sensibly designed language *exactly one* of the following statements has to be true:

1\) This is illegal.  
2\) E\<T\>.N calls C.M no matter what T is.  
3\) E\<C\>.N calls C.M but E\<D\>.N calls D.M .

As we've discussed, with static methods we chose the first option. But with instance methods, we choose the second\! Our earlier objection to it -- that the user clearly meant for the more derived method to be called -- melts away. Why? Because as far as we are concerned, that might as well have been public static void N(**C** t) { t.M(); } which you would reasonably expect to always call the less derived method, since its not virtual.

Why not \#3? Again, it has to do with static analysis. What it really comes down to is that in both the static and the instance cases, C.M and D.M are **entirely different methods**. The "new" calls that out; these are two different methods which just happen to share the same name. You can think of every method as having a "slot" in an object; in both the static and instance cases, we have defined two slots, not one. Had this been a virtual override then there would have been just one slot, and the *contents* of that slot would be determined at runtime. But in the non-virtual case there are *two* slots.

**When the compiler generates the generic code it resolves all the slots at compilation time.** **The jitter does not change them**. Indeed, the jitter does not know how\! The jitter has *no idea* that D.M has anything to do with C.M ; again, they are completely different methods that just coincidentally share a name. They have different slots so they are different methods.

I hope that all makes sense. Now to answer a few selected questions from the comments:

First, a number of readers noted that there are languages (Delphi, Smalltalk) which have implemented the concept of "class" methods. That is, methods which are not associated with an instance (so they are neither instance nor virtual), but nevertheless, which method is called is determined at runtime (so they are not static.) How the .NET versions of these languages do the codegen, I do not know; my guess would be that they emit code that does late binding via reflection or some similar mechanism.

Second, this raises the question of whether C\# ought to support some of the dynamic features which languages like JScript, Ruby, Python, etc, support. The VB team's motto as far as this question is concerned has always been *early binding when possible, late binding when necessary*.  The C\# team, by contrast, has always cleaved to the principle that we do *early binding when possible, late binding when the user explicitly writes umpteen dozen of lines of ugly reflection code.*

However, we do recognize both the power of dynamic language features, and the ugliness and pain of explicit reflection. We are considering ways to make late-bound code easier in C\# without making it nigh-indistinguishable from early-bound code (as VB sometimes does.)  These are very preliminary thoughts; we are still finishing up C\# 3.0 here; we've hardly begun thinking about C\# 4.0. But when we do, dynamic language features will be high on the list of things to think about. Anyone with bright ideas, by all means, send them my way.

Third, a number of readers noted that the word "static" has been consistently used in C\# to mean "associated with a class", and not "determined at compile time".  I agree; this is a good example of a poorly named abstraction. Language features should be named based on how the user is intended to use the feature, rather than on how we high-falutin' compiler designers classify different kinds of method calls. We allowed the "static" keyword to be used to mean "associated with a class". Why should "static" mean "associated with a class?"  That makes no sense at all. It's really an accident. But it is one we are stuck with now. Part of the reason why I write this blog is to dig into some of these weird historical corners and the associated unfortunate artefacts of less-than-pure design choices.

And fourth, completely off topic, a reader asks us to expose the internals of the compiler's abstract syntax tree. Again, we are in the very, very early stages of designing C\# 4.0, so **I cannot comment on specific features or timetables**. But I will say that **many people** have asked for this. I also note that in C\# 2.0 and 3.0, we concentrated almost entirely on adding language features, so much so that we have fallen behind somewhat on the tools support for them.

Exposing the internals of the compiler so that third parties could more easily create analysis-and-rewriting tools would be a great way to combine the power of the C\# semantic analysis engine with the power of community involvement. I encourage anyone who has ideas about what sort of tools they would like to develop, and what sort of API the compiler team could provide to make that easier, to drop me a note.  We need all the real-world usage cases we can get in order to decide how to invest our time and energy in the next version.

