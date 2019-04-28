# Future Breaking Changes, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/31/2007 10:00:00 AM

-----

[Last time](http://blogs.msdn.com/ericlippert/archive/2007/08/30/future-breaking-changes-part-one.aspx) I mentioned that one of the subtleties of programming language design is weighing the benefit of adding a feature against the pain it will cause you in the future. This is a specific subset of a more general set of problems. Languages run into the same problem that other large, multiply-versioned software products run into; if you have n existing features and want to add m new features, then you have at least m x n possible interactions to consider. In an ideal world the features would all be [orthogonal](http://blogs.msdn.com/ericlippert/archive/2005/10/28/483905.aspx) to each other, but in reality of course new features modify the behaviour of existing features.

Just as a random example, what on earth do [partial methods](http://blogs.msdn.com/wesdyer/archive/2007/05/23/in-case-you-haven-t-heard.aspx) have to do with [expression trees](http://www.interact-sw.co.uk/iangblog/2005/09/30/expressiontrees)? Both are new features for C\# 3.0, but other than that, they seem to have nothing to do with each other. But what happens when you have an expression tree lambda which contains *a call to a partial method which is going to be removed*? This interaction has to be carefully defined. *Every interaction between every feature has to be carefully defined*, and the larger the area of the feature interaction is, the more likely it is that there will be unfortunate consequences of this nonorthogonality, such as breaking changes.

I said last time that implicitly typed lambdas are just such a feature -- big surface area of nonorthogonal interactions which will lead to future breaking changes. I'd like to explore that in more detail, because it might not be immediately obvious why that is the case.

The rule for determining whether an [implicitly typed lambda](http://blogs.msdn.com/ericlippert/archive/2007/01/10/lambda-expressions-vs-anonymous-methods-part-one.aspx) is convertible to a given delegate type is (somewhat simplified) “If possible, infer the return type and the types of the lambda parameters from the delegate type. Try to bind the lambda with those types. If the inference and binding succeeds with no errors then the lambda expression is convertible to that delegate type.”

For example, if we had a lambda expression like x=\>x - 123.4, that would be convertible to [Func\<double, double\>](http://blogs.msdn.com/ericlippert/archive/2006/06/21/641831.aspx) but not to Func\<string, double\> (because the subtraction cannot be legally bound) or Func\<double, int\> (because the conversion to the return type cannot be bound).

Notice how the overlap between this new feature – implicitly typed lambda convertibility to a delegate type – overlaps with almost every other feature in the entire C\# language. Consider the impact of this design decision on the potential for breaking changes in the future. *Every single time we make a change to the rules for how the body of a method is bound, we are potentially changing whether a given lambda expression is convertible to a given type.*

Now, is that really so bad? Generally when we make changes to the method body binding rules, we do so in as non-breaking a manner as possible. We should never be making an existing lambda that *does* convert suddenly *stop* converting, because then we would also be breaking compilation of the equivalent *nominal* method body. Really we should only be causing presently-erroneous code to suddenly start being non-erroneous, and as we’ve already discussed, that’s not a breaking change.

Or is it?

Actually, now in a lot of cases it could be.

Consider a silly example. Suppose we decide that in some hypothetical C\# 4.0(‡) language it should be legal to subtract a double from a string. That’s not entirely farfetched – that’s perfectly legal in JScript. You just convert the string to a number and subtract. We might reason that this new feature is not a breaking change, because no program that ever subtracted a double from a string ever compiled before. But it *is* a breaking change, because now *this* program stops compiling:

 

using System;  
class Program {  
  void M(Func\<double, int\> f){}  
  void M(Func\<string, double\> f){}  
  void M(Func\<double, double\> f){}  
  static void Main() {  
    M(x=\>x-123.4);  
  }  
}

Before the change, the program unambiguously chooses the third overload. But with our change to addition semantics, now the second and third overloads would both work. Neither would be clearly better than the other. The compiler would then give an ambiguity error. Thus, this is a breaking change.

It gets even worse; with some cleverness we could come up with more subtle breaking changes, where the compiler would not produce an error but instead would choose a *different* overload than before. At least a program which fails to compile calls attention to the problem; recompiling and having the behaviour change subtly might go unnoticed for a long time.

This is a general problem with overload resolution: it depends upon conversion semantics. If we make the conversion semantics more strict then it is possible to go from having exactly one overload which works to zero, which is a breaking change. If we make the conversion semantics less strict then it is possible to go from having exactly one overload which works to two, which is also (usually) a breaking change.

Since lambda convertibility depends on every other language rule for binding an expression, any change to *any* of those rules is a potential change in convertibility, and hence a potential breaking change for overload resolution. Now, hopefully few real-world codebases will end up in the above situation, where a method is overloaded solely on a bunch of different delegate types, but still, I worry. We think implicitly typed lambdas are worth it, but this was a tough call that we agonized over for a long time.

Next time on FAIC: More on subtle language design issues involving breaking changes. Have a pleasant Labour Day weekend, Canadian and American readers\!

-----

(‡) I call out that this is the *hypothetical* C\# 4.0 compiler because of course I do not discuss the feature set of unannounced and non-existing products on public blogs.

