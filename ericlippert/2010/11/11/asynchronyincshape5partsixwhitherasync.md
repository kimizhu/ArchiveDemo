# Asynchrony in C\# 5 Part Six: Whither async?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/11/2010 6:54:00 AM

-----

A number of people have asked me what motivates the design decision to require any method that contains an "await" expression to be prefixed with the contextual keyword "async".

Like any design decision there are pros and cons here that have to be evaluated in the context of many different competing and incompossible principles. There's not going to be a slam-dunk solution here that meets every criterion or delights everyone. We're always looking for an attainable compromise, not for unattainable perfection. This design decision is a good example of that.

One of our key principles is "avoid breaking changes whenever reasonably possible". Ideally it would be nice if every program that used to work in C\# 1, 2, 3 and 4 worked in C\# 5 as well. (\*) As I mentioned [a few episodes back](http://blogs.msdn.com/b/ericlippert/archive/2010/09/27/ambiguous-optional-parentheses-part-three.aspx), (\*\*) when adding a prefix operator there are many possible points of ambiguity and we want to eliminate all of them. We considered many heuristics that could make good guesses about whether a given "await" was intended as an identifier rather than a keyword, and did not like any of them.

The heuristics for "var" and "dynamic" were much easier because "var" is only special in a local variable declaration and "dynamic" is only special in a context in which a type is legal. "await" as a keyword is legal almost everywhere inside a method body that an expression or type is legal, which greatly increases the number of points at which a reasonable heuristic has to be designed, implemented and tested. The heuristics discussed were **subtle** and **complicated**. For example, var x = y + await; clearly should treat await as an identifer but should var x = await + y do the same, or is that an await of the unary plus operator applied to y? var x = await t; should treat await as a keyword; should var x = await(t); do the same, or is that a call to a method called await?

Requiring "async" means that we can eliminate all backwards compatibility problems at once; any method that contains an await expression must be "new construction" code, not "old work" code, because "old work" code never had an async modifier.

An alternative approach that still avoids breaking changes is to use a two-word keyword for the await expression. That's what we did with "yield return". We considered many two-word patterns; my favourite was "wait for". We rejected options of the form "yield with", "yield wait" and so on because we felt that it would be too easily confused with the subtly different continuation behaviour of iterator blocks. We have effectively trained people that "yield" logically means "proffer up a value", rather than "cede flow of control back to the caller", though of course it means both\! We rejected options containing "return" and "continue" because they are too easily confused with those forms of control flow. Options containing "while" are also problematic; beginner programmers occasionally ask whether a "while" loop is exited the moment that the condition becomes false, or if it keeps going until the bottom of the loop. You can see how similar confusions could arise from use of "while" in asynchrony.

Of course "await" is problematic as well. Essentially the problem here is that there are two kinds of waiting. If you're in a waiting room at the hospital then you might wait by falling asleep until the doctor is available. Or, you might wait by reading a magazine, balancing a chequebook, calling your mother, doing a crossword puzzle, or whatever. The point of task-based asynchrony is to embrace the latter model of waiting: you want to keep getting stuff done on this thread while you're waiting for your task to complete, rather than sleeping, so you wait by remembering what you were doing, and then go do something else while you're waiting. I am hoping that the user education problem of clarifying which kind of waiting we're talking about is not insurmountable.

Ultimately, whether it is "await" or not, the designers really wanted it to be a single-word feature. We anticipate that this feature will potentially be used numerous times in a single method. Many iterator blocks contain only one or two yield returns, but there could be dozens of awaits in code which orchestrates a complex asynchronous operation. Having a succinct operator is important.

Of course, you don't want it to be *too* succinct. F\# uses "do\!" and "let\!" and so on for their asynchronous workflow operations. That\! makes\! the\! code\! look\! exciting\! but it is also a "secret code" that you have to know about to understand; it's not very discoverable. If you see "async" and "await" then at least you have some clue about what the keywords mean.

Another principle is "be consistent with other language features". We're being pulled in two directions here. On the one hand, you don't have to say "iterator" before a method which contains an iterator block. (If we had, then "yield return x;" could have been just "yield x;".) This seems inconsistent with iterator blocks. On the other hand... let's return to this point in a moment.

Another principle we consider is the "principle of least surprise". More specifically, that small changes should not have surprising nonlocal results. Consider the following:

void Frob\<X\>(Func\<X\> f) { ... }  
...  
Frob(()=\> {  
    if (whatever)  
    {  
        await something;  
        return 123;  
    }  
    return 345;  
  } );

It seems bizarre and confusing that commenting out the "await something;" changes the type inferred for X from Task\<int\> to int. We do not want to add return type annotations to lambdas. Therefore, we'll probably go with requiring "async" on lambdas that contain "await":

Frob(async ()=\> {  
    if (whatever)  
    {  
        await something;  
        return 123;  
    }  
    return 345;  
  } );

Now the type inferred for X is Task\<int\> even if the await is commented out.

That is strong pressure towards requiring "async" on lambdas. Since we want language features to be consistent, and it seems inconsistent to require "async" on anonymous functions but not on nominal methods, that is indirect pressure on requiring it on methods as well.

Another example of a small change causing a big difference:

 

Task\<object\> Foo()  
{  
    await blah;  
    return null;  
}

if "async" is not required then this method with the "await" produces a non-null task whose result is set to null. If we comment out the "await" for testing purposes, say, then it produces a null task -- completely different. If we require "async" then the method returns the same thing both ways.

Another design principle is that the stuff that comes before the body of a declared entity such as a method is all stuff that is represented in the *metadata* of the entity. The name, return type, type parameters, formal parameters, attributes, accessibility, static/instance/virtual/override/abstract/sealed-ness, and so on, are all part of the metadata of the method. "async" and "partial" are not, which seems inconsistent. Put another way: "async" is solely about describing the *implementation details* of the method; it has no impact on how the method is used. The caller cares not a bit whether a given method is marked as "async" or not, so why put it right there in the code where the person writing the caller is likely to read it? This is points against "async".

On the other hand, another important design principle is that interesting code should call attention to itself. Code is read a lot more than it is written. Async methods have a very different control flow than regular methods; it makes sense to call that out at the top where the code maintainer reads it immediately. Iterator blocks tend to be short; I don't think I've ever written an iterator block that does not fit on a page. It's pretty easy to glance at an iterator block and see the yield. One imagines that async methods could be long and the 'await' could be buried somewhere not immediately obvious. It's nice that you can see at a glance from the header that this method acts like a coroutine.

Another design principle that is important is "the language should be amenable to rich tools". Suppose we require "async". What errors might a user make? A user might have an have a method with the async modifier which contains no awaits, believing that it will run on another thread. Or the user might write a method that does have awaits but forget to give the "async" modifier. In both cases we can write code analyzers that identify the problem and produce rich diagnostics that can teach the developer how to use the feature. A diagnostic could, for instance, remind you that an async method with no awaits does not run on another thread and give suggestions for how to achieve parallelism if that's really what you want. Or a diagnostic could tell you that an int-returning method containing an await should be refactored (automatically, perhaps\!) into an async method that returns Task\<int\>. The diagnostic engine could also search for all the callers of this method and give advice on whether they in turn should be made async. If "async" is not required then we cannot easily detect or diagnose these sorts of problems.

That's a whole lot of pros and cons; after evaluating all of them, and lots of playing around with the prototype compiler to see how it felt, the C\# designers settled on requiring "async" on a method that contains an "await". I think that's a reasonable choice.

**Credits**: Many thanks to my colleague Lucian for his insights and his excellent summary of the detailed design notes which were the basis of this episode.

**Next time**: I want to talk a bit about exceptions and then take a break from async/await for a while. A dozen posts on the same topic in just a few weeks is a lot.

-----

(\*) We have violated this principle on numerous occasions, both (1) by accident, and (2) deliberately, when the benefit was truly compelling and the rate of breakage was likely to be low. The famous example of the latter is F(G\<A,B\>(7)). In C\# 1 that means that F has two arguments, both comparison operators. In C\# 2 that means F has one argument and G is a generic method of arity two.

(\*\*) When I wrote that article I knew that we would be adding "await" as a prefix operator. It was an easy article to write because we had recently gone through the process of noodling on the specification to find the possible points of ambiguity. Of course I could not use "await" as the example back in September because we did not want to telegraph the new C\# 5 feature, so I picked "frob" as nicely meaningless.

