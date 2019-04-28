# Never Say Never, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/21/2011 6:59:00 AM

-----

Can you find a lambda expression that can be implicitly converted to Func\<T\> for *any possible T?*

.

.

.

.

.

.

.

.

.

.

.

**Hint**: The same lambda is convertible to Action as well.

.

.

.

.

.

.

.

.

.

 

Func\<int\> function = () =\> { throw new Exception(); };

The rule for assigning lambdas to delegates that return int is not "the body must return an int". Rather, the rules are:

\* All returns in the block must return an expression convertible to int.  
\* The end point of the block must not be reachable.

Both those conditions are met. The first one is vacuously met; zero out of zero returns meet the condition, so that's all of them. The second one is met because the compiler can deduce that no possible code path hits that end brace. Either "new Exception()" throws, or it goes into an infinite loop, or it succeeds and its value is thrown; no matter what, there's no possibility of the function completing normally. Of course the conditions are met for any type argument, not just int.

Similarly, it's assignable to Action because the rule for Action is simply that every return in the block must not have an expression. Again, that condition is met vacuously.

The rule for lambdas is just a special case of the rule for regular functions. This is perfectly legal, for precisely the same reasons:

 

int I.M()  
{  
  throw new NotImplementedException();  
}

Why then is this application of the "extract method" refactoring not legal?

 

private static void AlwaysThrows()  
{  
  throw new NotImplementedException();  
}  
int I.M()  
{  
  AlwaysThrows();  
}

The problem here is that the C\# compiler does not perform interprocedural control flow analysis. We do analysis of one method body at a time, and we trust the declared return type of the method to be accurate. The declared return type of AlwaysThrows is void, and void means that it *returns no value*, but it does possibly *return*. Therefore, the end point of the call to AlwaysThrows is reachable, and therefore the end point of M is reachable without returning an integer. You and I both know that it is not reachable, but the compiler is not sophisticated enough to know that.

Of course, this is a silly example, but it doesn't take much to turn this into a realistic example. You see this sort of thing in unit testing frameworks all the time:

 

Frog frog;  
try  
{  
    frog = Animals.MakeFrog();  
}  
catch(Exception ex)  
{  
  LogAndThrowTestFailure(ex); // always throws  
}  
frog.Ribbit();

The compiler complains that frog.Ribbit() is illegal because MakeFrog might have thrown before frog was assigned, and LogAndThrowTestFailure -- which we know always throws but the *compiler* doesn't know that -- might have returned normally, in which case frog is not definitely assigned at the point of the call. If instead it had been

 

catch(Exception ex)  
{  
  throw LogTestFailureAndReturnAnotherException(ex);  
}

then the compiler would correctly reason that the call to Ribbit is only reachable if the assignment succeeded.

What, if anything, can we do about this?

In **practice**, nothing. You've got to write something like

 

int I.M()  
{  
  AlwaysThrows();  
  return 0;  
}

to shut the compiler up, or make AlwaysThrows return the exception and then throw it.

What about in **theory**? Is there anything the language designers could have done to ease this burden?

As I mentioned before, we could do interprocedural analysis, but in practice that gets real messy real fast. Imagine a hundred mutually recursive methods that all go into an infinite loop, throw, or call another method in the group. Designing a compiler that can logically deduce reachability from a complex topology of calls is doable, but potentially a lot of work. Also, interprocedural analysis only works if you have the source code for the procedures; what if one of these methods is in an assembly, and all we have to work with is the metadata? (Moreover, as we'll see next time, even interprocedural flow analysis is insufficient to solve the problem in general.)

What we need to solve this problem without interprocedural analysis of source code is a another kind of return type. The CLR supports three kinds of return types today. You can return **values** of value type or reference type, like int or string. You can return **nothing**, in which case the method is marked "void". Or you can return an **alias to a variable**. (C\# does not support this latter feature; C\# only supports "ref" on variables going *in* to a method call, but we could also support ref on variables coming *out* if we chose to. Don't hold your breath while waiting for it.) What we need is a fourth kind of return type, the "this method **never** returns normally" return type. Such a method would have to contain no returns whatsoever and not have a reachable end point. We could know that it does not have a reachable end point by checking whether on every possible code path it always throws, always goes into an infinite loop, or always calls another "never" method.

Some programming languages do have a "never" return type; [Curl](http://developers.curl.com/userdocs/docs/en/api-ref/never-returns.html), for example. A similar function annotation has also been proposed for ECMAScript. But since doing it properly in C\# requires support from the verifier in the CLR, it's unlikely that it will become a feature of mainstream CLR languages. Particularly when there are such easy workarounds for the rare circumstances in which you are calling a method that never returns. (\*)

**Next time**: could we be more clever? Just how clever can we be?

(\*) For additional thoughts on programming styles in which methods never return, see my long series of articles on [Continuation Passing Style](http://blogs.msdn.com/b/ericlippert/archive/tags/continuation+passing+style/).

