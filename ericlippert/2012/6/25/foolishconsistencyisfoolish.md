# Foolish consistency is foolish

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/25/2012 10:18:00 AM

-----

Once again today's posting is presented as a dialogue, as is my wont.

> **Why is var sometimes *required* on an implicitly-typed local variable and sometimes *illegal* on an implicitly typed local variable?**

That's a good question but can you make it more precise? Start by listing the situations in which an implicitly-typed local variable either must or must not use var.

> **Sure. An implicitly-typed local variable must be declared with var in the following statements:**
> 
> var x1 = whatever;  
> for(var x2 = whatever; ;) {}  
> using(var x3 = whatever) {}  
> foreach(var x4 in whatever) {}  
> 
> **And an implicitly-typed local variable must not be declared with var in the following expressions:**
> 
> from c in customers select c.Name  
> customers.Select(c =\> c.Name)
> 
> **In both cases it is not legal to put var before c, though it would be legal to say:**
> 
> from Customer c in customers select c.Name  
> customers.Select((Customer c) =\> c.Name)
> 
> **Why is that?**

Well, let me delay answering that by criticizing your question further. In the query expression and lambda expression cases, are those in fact implicitly typed locals in the first place?

> **Hmm, you're right; technically neither of those cases have local variables. In the lambda case, that is a formal parameter. But a formal parameter behaves almost exactly like a local variable, so it seems reasonable to conflate the two in casual conversation. In the query expression, the compiler is going to syntactically transform the range variable into an untyped lambda formal parameter regardless of whether the range variable is typed or not.**

Can you expand on that last point a bit?

> **Sure. When you say**
> 
> from Customer c in customers select c.Name
> 
> **that is not transformed by the compiler into**
> 
> customers.Select((Customer c) =\> c.Name)
> 
> **Rather, it is transformed into**
> 
> customers.Cast\<Customer\>().Select(c =\> c.Name)

Correct. Discussing why that is might be better left for another day.

> **Indeed; the point here is that regardless of whether a type appears in the query expression, the lambda expression in the transformed code is going to have an untyped formal parameter.**

So now that we've clarified the situation, what is your question?

> **C\# is inconsistent; var is *required* on an implicitly-typed local variable (regardless of the "container" of the local declaration), but var is *illegal* on an implicitly-typed lambda parameter (regardless of whether the lambda parameter is "natural" or is the result of a query expression transformation). Why is that?**

You keep on asking "why?" questions, which I find difficult to answer because your question is secretly a "why not?" question. That is, the question you really mean to ask is "*I have a notion of how the language obviously ought to have been designed; why is it not like that?*"  But since I do not know what your notion is, it's hard to compare its pros and cons to the actual feature that you find inconsistent.

The problem you are raising is one of inconsistency; I agree that you have identified a bona fide inconsistency, and I agree that a foolish, unnecessary inconsistency is bad design. Our language is designed to be easy to comprehend; foolish inconsistencies work against comprehensibility. But what I don't understand is how you think that inconsistency ought to have been addressed.

I can see three ways to address that inconsistency. First, make var *required* on lambdas and query expressions, so that it is consistently required. Second, make var *illegal* on all locals, so that it is consistently illegal. And third, make it *optional* everywhere. What is the real "why not?" question?

> **You're right; I've identified an inconsistency but have not described how I think that inconsistency could be removed. I don't know what my real "why not?" is so let's look at all of them; what are the pros and cons of each?**

Let's start by considering the first: require var everywhere. That would then mean that you have to write:

from var c in customers join var o in orders...

instead of

from c in customers join o in orders...

And you have to write

customers.Select((var c) =\> c.Name)

instead of

customers.Select(c =\> c.Name)

This seems clunky. What function does adding var in these locations serve? It does not make the code any more readable; it makes it less readable. We have purchased consistency at a high cost here. The first option seems untenable.

Now consider the second option: make var illegal on all locals. That means that your earlier uses of var on locals would become:

x1 = whatever;  
for(x2 = whatever; ;) {}  
using(x3 = whatever) {}  
foreach(x4 in whatever) {}

The last case presents no problem; we know that the variable declared in a foreach loop is always a new local variable. But in the other three cases we have just added a new feature to the language; we now have not just implicitly *typed* locals, we now have implicitly *declared* locals. Now all you need to do to declare a new local is to assign to an identifier you haven't used before.

There are plenty of languages with implicitly declared locals, but it seems like a very "un-C\#-ish" feature. That's the sort of thing we see in languages like VB and VBScript, and even in them you have to make sure that Option Explicit is turned off. The price we pay for consistency here is very different, but it is still very high. I don't think we want to pay that price.

The third option -- make var optional everywhere-- is just a special case of the second option; again, we end up introducing implicitly declared locals.

Design is the art of compromising amongst various incompatible design goals. In this case, consistency is a goal but it loses to more practical concerns. This would be a foolish consistency.

> **Is the fact that there is no good way out of this inconsistency an indication that there is a deeper design problem here?**

Yes. When I gave three options for removing the inconsistency, you'll notice that I made certain assumptions about what was and was not allowed to change. If we were willing to make larger changes, or if we had made different design decisions in C\# 1.0, then we wouldn't be in this mess in the first place. The deeper design problem here is that the fact that a local variable declaration has the form **

*type identifier **;***

This is not a great statement syntax in the first place. Suppose C\# 1.0 had instead said that a local variable must be declared like this:

***var** identifier **:** type **;***

> **I see where you are going; JScript.NET does use that syntax, and makes the type annotation clause optional. And of course Visual Basic uses the moral equivalent of that syntax, with Dim instead of var and As instead of :.**

That's right. So in typing-required C\# 1.0 we would have

var x1 : int = whatever;  
for(var x2 : int = whatever; ;) {}  
using(var x3 : IDisposable = whatever) {}  
foreach(var x4 : int in whatever) {}

This is somewhat more verbose, yes. But this is very easy to parse, both by the compiler and the reader, and is probably more clear to the novice reader. *It is crystal clear that a new local variable is being introduced by a statement*. And it is also nicely reminiscent of base class declaration, which is a logically similar annotation. (You could have just as easily asked "*Why does the constraining type come to the left of the identifier in a local, parameter, field, property, event, method and indexer, and to the right of the identifier in a class, struct, interface and type parameter?*" Inconsistencies abound\!)

Then in C\# 3.0 we could introduce implicitly typed locals *by simply making the annotation portion optional*, and allowing all of the following statements and expressions:

var x1 = whatever;  
for(var x2  = whatever; ;) {}  
using(var x3  = whatever) {}  
foreach(var x4 in whatever) {}  

from c in customers select c.Name  
from c : Customer in customers select c.Name  
customers.Select(c =\> c.Name)  
customers.Select((c : Customer) =\> c.Name)

> **If this syntax has nicer properties then why didn't you go with it in C\# 1.0?**

Because we wanted to be familiar to users of C and C++. So here we have one kind of consistency -- consistency of experience for C programmers -- leading a decade later to a problem of C\# self-consistency.

The moral of the story is: good design requires being either impossibly far-sighted, or accepting that you're going to have to live with some small inconsistencies as a language evolves over decades.

