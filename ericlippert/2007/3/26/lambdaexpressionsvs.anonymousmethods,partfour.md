# Lambda Expressions vs. Anonymous Methods, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/26/2007 7:05:00 PM

-----

Hey all, sorry for the long time between posts; I have been crazy busy [recruiting](http://blogs.msdn.com/ericlippert/archive/2007/01/30/free-food-and-meet-the-compiler-guy-and-win-an-xbox-360.aspx), interviewing, fixing bugs, making performance improvements and implementing last-minute changes to the language and expression tree library. The last few posts about lambda binding yielded many interesting comments which I hope to address over the next month or so.

Before that though -- back when I [started](http://blogs.msdn.com/ericlippert/archive/2007/01/10/lambda-expressions-vs-anonymous-methods-part-one.aspx) this series of posts I noted that lambda expressions have an interesting property. Namely, *the semantic analysis of the lambda body may depend upon what type the lambda is being converted to*. This seemingly trivial fact has an important impact upon the compiler performance during analysis. At long last, let me demonstrate the impact this fact has.

Consider the following set of overloads:

int M(Func\<int, int\> f){ /\* whatever ... \*/ }  
string M(Func\<string, string\> f){ /\* whatever ... \*/ }  

Now suppose we have a call with a lambda argument:

M(x=\>x.ToUpper()); 

During overload resolution we make a list of all the potential overloads and we see which are applicable. That is, which overloads have the property that all of the arguments are convertible to the types of their corresponding formal parameters. Then of all the applicable methods, we pick the best match if there happens to be more than one.

Therefore we must try this binding two ways. We try it as though it were

M((int x)=\>x.ToUpper()); 

and

M((string x)=\>x.ToUpper()); 

The former produces an error while binding the body because there is no ToUpper method on int. The latter binds the body correctly and the return type matches the expected return type, so this one succeeds. The first M is not applicable so it is discarded, the second one is applicable, so it is chosen, all is good in the world.

Or is it?

What if we did something crazy like this?

M(x1=\>M(x2=\>x1.ToLower() + x2.ToUpper()); 

OK, so now what do we do? We try to do overload resolution on the outer call. Again, we must try both int and string as candidates for x1. But then when we get to the outer body, we are faced with the same problem for x2.

We have to try all four possibilities for {x1, x2} (that is, {int, int}, {int, string}, {string, int} and {string, string}) in order to determine how many of them work, and of the ones that work, which one is the best. In this particular example "both are string" is the only one that works. (We could with a little more cleverness come up with examples where there were multiple possible matches that worked.)

So perhaps you see where this is going. If there are m overloads that are nested in this manner n deep, then the compiler must try m<sup>n</sup> possibilities in order to determine which is the unique best one. If it's 2<sup>2</sup> = 4, that's not a big deal. If we triple each and go to 6<sup>6</sup> = 46656, that's rather a lot of binding to do, and it takes up rather a lot of time and memory.

Does this seem like a contrived example? Unfortunately, it is not. In the original prototype of C\# 3.0, nested from clauses in queries were translated into nested lambdas in *exactly* this manner, and most of the translation methods have at least two overloads, some have many more. We were easily getting into situations where simple queries could involve *millions* of body bindings. Fortunately we realized the problem in time; my colleague [Wes](http://blogs.msdn.com/wesdyer/) came up with an alternative translation strategy which does not generate nested lambdas so aggressively and we therefore avoid this problem in the common case. You can still get into it yourself though; please do not. 

Next time I'll create a truly contrived (and rather hilarious) example which demonstrates just how bad things can get and just how hard you can make the compiler work to do overload resolution on lambdas.

