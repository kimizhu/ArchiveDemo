# What is "binding" and what makes it late?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/6/2012 10:39:00 AM

-----

"Late binding" is one of those computer-sciency terms that, like "strong typing", means different things to different people. I thought I might describe what the term means to me.

First off, what is "binding"? We can't understand what it means to bind late if we don't know what it is to bind at all.

A compiler is by definition a device which takes in a text written in one language and outputs code that "means the same thing" in another language. The compiler I work on, for example, takes in C\# and outputs CIL(\*). All the principle tasks performed by a compiler falls into one of three broad buckets:

  - Syntactic analysis of the input text
  - Semantic analysis of the syntax
  - Generation of the output text -- we will be unconcerned with this step in this article

Syntactic analysis is analysis of the input text that does not consider anything about the *meaning* of that text; the syntactic analyzer is concerned with first determining the *lexical* structure of the program (that is, where all the boundaries are between comments, identifiers, operators, and so on) and then from that lexical structure determining the *grammatical* structure of the program: where are the boundaries between classes, methods, statements, expressions, and so on.

Semantic analysis then takes the results of the syntactic analysis and associates meanings with various syntactic elements. For example, when you say:

class X {}  
class B {}  
class D : B  
{  
  public static void X() { }  
  public static void Y() { X(); }  
}

syntactic analysis determines that there are three classes, that one of them contains two methods, the second method contains a statement which is a call expression. Semantic analysis determines that the X in X(); refers to the method D.X(), rather than, say, the type X declared above. That's an example of "binding" in its most widely-agreed-upon sense: **binding is the association of a syntactic element that names a method with a logical element of the program**.

"Binding" in the sense of "early" or "late" binding is almost always used to refer to the evaluation of a name used as a method call. That, however, is far too strict a definition of "binding" for me. I would also use "binding" to describe how the compiler's semantic analyzer determines that class D inherits from class B; the name "B" is bound to the class.

Moreover, I would use "binding" to describe other analyses as well. If you had an expression 1 \* 2 + 1.0 in your program then I would say that the "+" operator is "bound" to the built-in operator that takes two doubles, adds them, and returns a third. People do not normally think of that analysis as associating the name "+" with a method, but nevertheless I consider it to be "binding".

Speaking even less carefully, I might also use "binding" to describe the association of types with expressions that do not directly name the type. In the example above if speaking casually I might say that the syntax 1 \* 2 is "bound" to the type int, even though obviously it does not name that type. The syntactic expression is strongly associated with that semantic element, even though it does not name it.

So, speaking generally I would say that "**binding" is any association of some fragment of syntax with some logical program element**. (\*\*)

What then is "late" vs "early" binding? People talk about these two kinds of bindings as though it was a binary choice, either early or late. As we'll see, actually there is more of a spectrum than you might imagine; some bindings are entirely early, some are partially early and partially late, and some are very late indeed. But before we get into that, earlier or later *than what*?

Basically by "early binding" we mean "the binding analysis is performed by the compiler and baked in to the generated program"; if the binding fails then the program does not run because the compiler did not get to the code generation phase. By "late binding" we mean "some aspect of the binding will be performed by the runtime" and therefore a binding failure will manifest as a runtime failure. Early and late binding might better be called "static binding" and "dynamic binding"; static binding is binding performed using "static" facts known to the compiler, and dynamic binding is performed using facts "dynamically" known to the runtime.

So which is better? Clearly neither is unambiguously better than the other; if one was clearly the winner then we wouldn't be having this discussion in the first place. The benefit of early binding is that you gain confidence that your program will not fail at runtime; the down side is that you lose the flexibility that comes with late binding. Early binding requires that all information required to make the right binding decision be known before the program runs; but that information might not be available until runtime.

I said that early and late binding falls on a spectrum. Let's take a look at some examples in C\# of how we can "turn the dial" on early-to-late binding.

We begin with our example above of calling the static method X. This analysis is entirely, unambiguously early. There is no doubt at all that when Y is called, it is going to call the method D.X. No part of that analysis is deferred until runtime and the call will unambiguously succeed.

Next, consider:

class B  
{  
  public void M(double x) {}  
  public void M(int x) {}  
}  
class C  
{  
  public static void X(B b, int d) { b.M(d); }  
}

Now we know less. We do a lot of early binding here; we know that b is of type B and that the call is to B.M(int). But unlike the previous example, we have no guarantee from the compiler that this invocation will succeed. b could be null. We are essentially deferring the analysis of whether or not the receiver of the call is valid to the runtime to determine. One normally does not think of that as a "binding" decision though, because we are not *associating syntax with a program element*. Let's make the call in C; a little more late bound by changing B:

class B  
{  
  public virtual void M(double x) {}  
  public virtual void M(int x) {}  
}

We are now doing some binding analysis at compile time; we know that the invocation will be on the virtual method declared by B.M(int). We know that the call will succeed in the sense that there will be such a method to call. However, we do not know which method will actually be called at runtime\! There could be a derived type that overrides that method; some completely other code could be called that appears nowhere in this program. **Virtual dispatch is a form of late binding;** the decision about which method to associate with the syntax b.M(d) is partially made by the compiler and partially made by the runtime.

How about this?

class C  
{  
  public static void X(B b, dynamic d) { b.M(d); }  
}

Now the binding is almost entirely deferred until runtime. The compiler generates code that tells the Dynamic Language Runtime that static analysis has determined that the static type of b is B and that the method being invoked is called M, but the actual overload resolution to determine whether it is B.M(int) or B.M(double) (or neither, if d is, say, a string) is done by the runtime based on that information. (\*\*\*)

class C  
{  
  public static void X(dynamic b, dynamic d) { b.M(d); }  
}

Now the only portion of the binding performed early is the fact that this is a method call on a method named M. This is almost as late-bound as it gets, but we can in fact go one step further:

class C  
{  
  public static void X(object b, object\[\] d, string m, BindingFlags f)  
  {  
    b.GetType().GetMethod(m, f).Invoke(b, d);  
  }  
}

Now everything is late-bound; we don't even know *what name* we are going to be associating with a method. All we can possibly know is that the author of X expects that the object passed as b has a single method named by the string in m that match the flags in f, and that it takes the arguments given in d. There is nothing whatsoever we can do to analyze this call site at compile time. (\*\*\*\*)

-----

(\*) Of course the output is encoded into a machine-readable binary format, rather than the human-readable CIL format.

(\*\*) You might then ask whether "binding" and "semantic analysis" are not synonyms; surely semantic analysis is nothing more than the association of syntactic elements with their meanings\! Binding is much of the semantic analysis phase of the compiler, but there are many analyses that we must perform after a method body is entirely "bound". Definite assignment analysis, for example, cannot by any reasonable stretch be called "binding"; it is not associating syntactic fragments with specific program elements. Rather, it is associating lexical *locations* with facts about program elements, facts like "the local variable blah is not definitely assigned at the beginning of this block". Similarly, optimizing arithmetic expressions is a form of semantic analysis but is clearly not "binding".

(\*\*\*) The compiler could still do lots of static analysis here. For example, suppose B were a sealed class with no methods named M. Even with a dynamic argument to the call we could know statically that the runtime binding of M will always fail, and tell you that at compile time. And in fact the compiler does some such analyses; precisely how it does so is a good topic for another day.

(\*\*\*\*) In some sense this example gives a counterexample to my definition of binding; we are no longer even associating a *syntactic element* with a method; we're associating the contents of a string with a method.

