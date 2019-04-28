# Lambda Expressions vs. Anonymous Methods, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/10/2007 12:53:00 PM

-----

As you know by now if you've been reading this blog for a while, I am incredibly excited about adding lambda expressions to C\# 3.0. I thought I'd talk a bit today about an incredibly subtle difference between C\# 2.0 anonymous methods and C\# 3.0 lambda expressions which has caused me no end of stress over the last year.

(I'd like to emphasize before I continue that none of this stuff is necessary to understand in order to use lambdas in C\# 3.0. The point of this article is not to describe anything that you need to know, but rather to give you a bit of a behind-the-scenes look at what some of the difficult issues are in language design and implementation.)

At first glance, lambda methods look like nothing more than a syntactic sugar, a more compact and pleasant syntax for embedding an anonymous method in code. Compare:

Func\<int, int\> f1 = delegate(int i) { return i + 1; };  
Func\<int, int\> f2 = i=\>i+1; 

The latter gets rid of the kludgy delegate keyword, it uses the "goes to" arrow familiar to math students, the type of the argument is no longer redundantly specified twice, and the braces and return keyword are elided. A very pleasant little syntactic sugar, but nothing else interesting is going on here, right?

Wrong. Being able to infer the parameter types from the conversion target type leads to a subtle but deep difference for the compiler implementer, ie, me. To see why, let's look at some more examples.

Both of these are errors:

Func\<int, int\> f3 = delegate(int i) { return i.ToString(); };  
Func\<int, int\> f4 = i=\>i.ToString(); 

And in fact both are the same error. Since the return value is not convertible to the return type of the delegate, neither the anonymous method nor the lambda expression are convertible to the delegate type.

Suppose we have a method bool M1(short s). Both of these are errors as well:

Func\<int, string\> f5 = delegate(int i) { return M1(i) ? i.ToString() : ""; };  
Func\<int, string\> f6 = i=\>M1(i) ? i.ToString() : ""; 

These are different errors\! The anonymous method's parameter types and return type match the delegate, so the anonymous method is implicitly convertible to the delegate type. However, i is not implicitly convertible to short, so the anonymous method body contains an error. **But the anonymous method itself is convertible to its target type.**

The lambda version contains the same error. Because it contains an error it is furthermore not convertible to the target type. Notice that it *would* be convertible to Func\<short, string\>.

So what's the big deal? We might give a slightly different error message here, but so what?

The problem is that since we do not know the types of the parameters until the target type is determined, it means that we cannot aggressively bind (by "bind" I mean "do full semantic analysis") the body of the lambda when the binder encounters the lambda. Rather, we have to put the lambda aside and say "come back to this thing later when we know what the target type is". In C\# 2.0 anonymous method bodies were bound eagerly because we always had enough information to determine if there was an error inside the anonymous method even if we didn't know the target type. We could bind the body first, and then later on double-check during convertibility checking to make sure that the parameter types and return type were compatible with the delegate. Every expression type in the compiler worked this way: you do a full analysis of the expression, and then you see if it is compatible with the type that it is being converted to.

With lambdas, the information flows in the opposite direction through the binder; first we have to know where we're going, and that then influences how the body is bound during the convertability checking.

This may still seem like an academic point. Next time I'll describe how this difference leads to a potentially serious (though hopefully highly unlikely in the real world) performance issue baked into the language semantics.

