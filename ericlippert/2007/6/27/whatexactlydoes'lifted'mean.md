# What exactly does 'lifted' mean?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/27/2007 10:00:00 AM

-----

(Note: all type placeholders S, T, U that I mention in this article are non-nullable value types.)

I got a question the other day from a reader who had been taking a close look at the C\# 2.0 standard. He noticed that when we talk about nullables in the standard we talk about "lifting", but we do so inconsistently. To summarize what the standard says:

1\) For every equality and relational operator that compares an S to a T and returns a bool there is a corresponding **lifted** operator that compares an S? to a T? and returns a bool. It returns false if either argument is null. (With the additional exception that if one argument to an equality operator is the null literal then the result may be determined by checking if the nullable term has a value.)

2\) For the unary operators + ++ - -- \! ~ where the operator takes an S and returns a T there is a corresponding **lifted** operator that takes an S? and returns a T?. It returns null if its argument is null.

3\) For the binary operators + - \* / % & | ^ \<\< \>\>, where the operator takes an S and a T and returns a U there is a corresponding **lifted** operator that takes an S? and a T? and returns a U?. It returns null if either argument is null.

4\) For user-defined conversions from S to T, there is a corresponding **lifted** user-defined conversion from S? to T?. It returns null if the argument is null.

5\) For built-in conversions from S to T, there are corresponding **nullable** conversions:  
5.1) from S? to T?, which returns null if the argument is null.  
5.2) from S to T?, which never returns null, and  
5.3) an explicit-only conversion from S? to T which throws if the argument is null.

Why does the standard call out that the conversions in (5) are "nullable" but not "lifted"? Everything else is "lifted"\!

This is a bit of a mess I'm afraid.

I talked to [Dr. T](http://blogs.msdn.com/madst/default.aspx) (AKA Mads Torgersen, the C\# language PM) about this. He defined for me precisely what mathematicians mean by “lifted”.

Suppose we've got a function f which maps values from a set A into a set B. That is f:A→B.

Suppose further that null is not a member of either A or B.

Now consider the sets A' = A ∪ { null } and B' = B ∪ { null }

We define the "lifted function" f' as

f':A'→B' such that f'(x) = f(x) for all x ∈ A and f'(null) = null

Similarly, if we had a two-argument function f: A × B → C, we would define f': A' × B' → C' as f'(x,y) = f(x,y) for all (x,y) ∈ A × B and null if either x or y is null.

What we’re getting at here is that “lifted” means “takes nulls, always agrees with the unlifted version when arguments are not null, maps everything else onto null”.

Now you probably see why I said this was a bit of a mess. By the mathematician's definition, (1) is incorrectly called "lifted", (2), (3), and (4) are correctly called "lifted", (5.1) is incorrectly NOT called "lifted", and (5.2) and (5.3) are correctly not called "lifted". Of course we can choose to define what "lifted" means *in C\#* any way we like, but it would be nice if our definition agreed with convention and was used consistently\!

I regret the confusion. I do not believe there is any particular sensible reason for these inconsistencies. Rather, the details were changed so many times over the years as the nullable feature was developed that these sorts of subtle problems crept into the spec and were never expunged. Though of course all of us have as a goal that the standard be a model of clarity and permanence, it is fundamentally a working, evolving, imperfect document; these kinds of things will happen. Hopefully in the next version of the standard some of these sorts of details will be tidied up.

I hope that answers the question\!

