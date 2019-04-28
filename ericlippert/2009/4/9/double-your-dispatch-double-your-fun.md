<div id="page">

# Double Your Dispatch, Double Your Fun

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/9/2009 11:50:00 AM

-----

<div id="content">

<div class="mine">

Here’s an interesting question I got the other day:

> 
> 
> <div class="quote">
> 
> If you have an overloaded operator == then any call to the operator method is “early bound” at *compile time* according to the *compile-time* types of the *operands*. But calling Equals() on an object is a virtual call; the actual method called is bound at *runtime* according to the *runtime type* of the *receiver*. This difference seems weird to me. What’s the relevant design principle here?
> 
> </div>

The short answer is that language designers and framework designers have different ways of looking at problems.  Overloaded operator == is a language convention, method Equals() is a framework convention. They’re different because their designers thought about the problem differently, had different user requirements, and had different tools available to solve the problem. The long answer is that **the whole thing is weird and neither works the way it ideally ought to.** Before I go on, I need a quick definition of what we mean by different kinds of method dispatching: With **nonvirtual dispatch**, overload resolution “statically” determines exactly which method is called based on the compile-time types of the arguments and the compile-time type of the receiver. **Virtual dispatch** statically determines which “method slot” is called based on the compile-time types of the arguments and the compile-time type of the receiver, but determines which actual method is called at runtime, based on the contents of the slot. The contents of the slot depend on the runtime type of the receiver. This is what we'll call “**single-virtual dispatch**”. That is, we are **considering only the runtime type of the *receiver***. The runtime types of the *arguments* are not relevant. You could imagine a language in which they were relevant by making up a new “multi-virtual" dispatch:<span class="code"> </span>

class B  
{  
  public **multivirtual** void M(object x) { }  
} class D : B  
{  
  public **multioverride** void M(object x){ }  
  public **multioverride** void M(string x){ }  
} \[...\]     object argument = "hello";  
    B receiver = new D();   
    receiver.M(argument);

In a single-virtual dispatch language, this would call the *first override in D* because only the runtime type of "receiver" is relevant when dispatching the call. In a multi-virtual language we could do something that we'll call “**double-virtual dispatch**” here, actually dispatching the call to the *second* *override in D*, by examining the runtime types of the receiver *and* the argument. Of course, you need not imagine it if you’d rather just see it for yourself. There are languages which support "multimethods", that is, methods that do single-, double-, triple-, and so on, virtual dispatch calls. [Dylan](http://en.wikipedia.org/wiki/Dylan_\(programming_language\)) comes immediately to mind, though there are many others. However, the CLR has “baked in” the mechanisms for efficient single dispatch and has not baked in the mechanisms we’d need for efficient multiple dispatch. (If you really want multiple dispatch, well, in C\# 4 you can use “dynamic”, and we’ll do **all** the usual compile-time analysis at runtime. That’s not very efficient compared to the one pointer indirection that is single dispatch, but it gets the job done and the performance is reasonable in many cases.) What does this have to do with equality? To answer the question posed, first we need to answer "why is Equals single-dispatch virtual?" Good question\! **In an ideal world it would be double- virtual dispatch\!** When you compare two objects for equality with the Equals method, **surely the type of the argument is every bit as relevant as that of the receiver**\! We all conceive of equality as a *symmetric* operation, but **Equals is deeply asymmetric**. The left side of the Equals, the receiver, gets special treatment that the right side, the argument, does not. That’s why most non-trivial implementations of Equals end up doing their own faked up double dispatch logic. You drive to the most derived implementation of the Equals operator available on the left side via single dispatch, and then that method is responsible for figuring out what the runtime type of the right side is, and doing the correct thing, if it cares. This is a simpler version of a more general pattern; double dispatch is often implemented on top of single dispatch when implementing the **visitor pattern**. (Do some web searches on that if you're interested in seeing ways to do double dispatch in single dispatch languages.) So, now think about this all from the point of view of the framework designer. You are designing from scratch an object-oriented class library and you want all objects to be able to be comparable for equality with all other objects. You have four choices: 1. Implement double dispatch in the CLR and make Equals double-virtual dispatch. Require all languages to understand the CLR’s double dispatch logic and generate calls to Equals correctly. 2. Make the method single-virtual dispatch, introducing an asymmetry. Let the developer sort it out if they want double dispatch by writing custom code, which they are doing *already* if they are overloading Equals, so it’s not much more work for them. 3. Make the method nonvirtual. *Always* call System.Object’s implementation, which just compares the bits of the two sides. 4. Cut the feature. None of these are clear wins – design, after all, is the art of making good compromises. From the framework designer’s point of view, (2) is the sweet spot. No huge architectural redesign of the CLR, but with maximum flexibility for the developer, at the price of making them do some work. And it’s “pay for play” work – developers who do not care can just use the default implementation and never worry about it. This is not so bad, all things considered. Lexically, a call to x.Equals(y) looks like a virtual method call, and therefore we are likely to think about it as a single-virtual dispatch call, like every other virtual call. But using the == operator is a whole other ball of wax. Users have the reasonable assumption that the == operator is symmetric. That thing does not look like a method call. Think about == from the point of view of the language designer. You want to allow overloading the equality operator to call a method when the operator is used. You have four choices: (1) Implement a compiler-generated double dispatch system for operators. The code generation is highly complex and the operations we need to do have no particularly efficient representation in the CLR. (2) Implement a single dispatch system that breaks users' reasonable expectations of symmetry and provides no value at all over simply calling Equals, aside from the minor improvement in syntax at the call site. Essentially this is no better than just making "x == y" a sugar for "x.Equals(y)". (3) Implement symmetric nonvirtual dispatch to a static method based on the compile-time types of the operands; give a compile-time error if a symmetric analysis uncovers an ambiguity. Let the static methods do whatever runtime dispatch they want, if the developer wants to implement that. After all, they’re already making a custom method. (4) Cut the feature. For the language designer, choice (3) is much more compelling than any of the others. So that's what we do.

The analysis of the equality operator in C\# is symmetric; we search the type hierarchies of *both* operands for operator overloads, throw them all into the pool, and then attempt to identify a “best” member of the pool. If we cannot find a unique best operator then we give an error.

In C\# 4.0, if the operands to == are "dynamic" then we will do the usual compile-time analysis *at runtime* based on the *runtime types*, which *effectively* does give you something like double-virtual dispatch, at the cost of running the compiler at runtime. We will cache the results of the analysis, so the second time you hit the call site, it should be reasonably efficient.

It is rather unfortunate that we give you two different ways to do what is arguably one thing. I personally would have been fine with having no Equals method on System.Object at all, and just let every language sort out what it means to compare two objects for equality. But then the argument is that equality is a concept which has scope beyond any one language and that objects should be in charge of determining their equality regardless of what any language has to say about the matter. It’s tricky to know where to draw that line.

</div>

</div>

</div>

