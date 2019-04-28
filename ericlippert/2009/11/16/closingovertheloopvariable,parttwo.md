# Closing over the loop variable, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/16/2009 8:18:00 AM

-----

(This is part two of a two-part series on the loop-variable-closure problem. [Part one is here](http://blogs.msdn.com/b/ericlippert/archive/2009/11/12/closing-over-the-loop-variable-considered-harmful.aspx).)

-----

**UPDATE**: We **are** taking the breaking change. In C\# 5, the loop variable of a foreach will be logically inside the loop, and therefore **closures will close over a fresh copy of the variable each time**. The "for" loop will not be changed. We return you now to our original article.

-----

Thanks to everyone who left thoughtful and insightful comments on last week's post.

More countries really ought to implement Instant Runoff Voting; it would certainly appeal to the geek crowd. Many people left complex opinions of the form "I'd prefer to make the change, but if you can't do that then make it a warning". Or "don't make the change, do make it a warning", and so on. But what I can deduce from reading the comments is that there is a general lack of consensus on what the right thing to do here is. In fact, I just did a quick tally:

Commenters who expressed support for a warning: 26  
Commenters who expressed the sentiment "it's better to not make the change": 24  
Commenters who expressed the sentiment "it's better to make the change": 25

Wow. I guess we'll flip a coin. :-)    (\*)

Four people suggested to actually make it an error to do this. That's a pretty big breaking change, particularly since we would be breaking not just "already broken" code, but plenty of code that works perfectly well today -- see below. That's not likely to happen.

People also left a number of interesting suggestions. I thought I'd discuss some of those a little bit.

First off, I want to emphasize that what we're attempting to address here is the problem that **the language encourages people to write code that has different semantics than they think it has**. The problem is NOT that the language has no way to express the desired semantics; clearly it does. Just introduce a new variable explicitly into the loop.

A number of suggestions were for ways that the language could more elegantly express that notion. Some of the suggestions:

 

foreach(var x in c) **inner**  
**foreachnew**(var x in c)  
foreach(**new** var x in c)  
foreach(var x **from** c)  
foreach(var x **inside** c)

Though we could do any of those, none of them by themselves solve the problem at hand. Today, you have to know to use a particular pattern with foreach to get the semantics you want: declare a variable inside the loop. With one of these changes, you still have to know to use a particular keyword to get the semantics you want, and it is still easy to accidentally do the wrong thing.

Furthermore, a change so small and so targetted at such a narrow scenario probably does not provide enough benefit to justify the large cost of creating a new syntax, particularly one which is still easily confused with an existing syntax.

C++ luminary Herb Sutter happened to be in town and was kind enough to stop by my office to describe to me how they are solving a related problem in C++. Apparently the next version of the C++ standard will include lambdas, and they're doing this:

 

*\[q, \&r\]* **(int x) -\> int** { return M(x, q, r); }

This means that the lambda captures outer variable q by value, captures r by reference, takes an int and returns an int. Whether the lambda captures values or references is controllable\! An interesting approach but one that doesn't immediately solve our problem here; we cannot make lambdas capture by value by default without a huge breaking change. Capturing by value would have to require new syntax, and then we're in the same boat again: the user has to know to use the new syntax when in a foreach loop.

A number of people also asked what the down sides of adding a warning are. The down side is that **a warning which warns about correct behaviour is a very bad warning**; it makes people change working code, and frequently they break working code in order to eliminate a warning that shouldn't have been present in the first place. Consider:

 

foreach(var insect in insects)  
{  
  var query = frogs.Where(frog=\>frog.Eats(insect));  
  Console.WriteLine("{0} is eaten by {1} frogs.", insect, query.Count());  
}

This makes a lambda closed over insect; the lambda never escapes the loop, so there's no problem here. But the compiler doesn't know that. The compiler sees that the lambda is being passed to a method called Where, and Where is allowed to do anything with that delegate, including storing it away to be called later. Which is exactly what Where does\! Where stores away the lambda into a monad that represents the execution of the query. The fact that the **query object doesn't survive the loop** is what keeps this safe. But how is the compiler supposed to suss out that tortuous chain of reasoning? We'd have to give a warning for this case, even though it is perfectly safe.

It gets worse. A lot of people are required by their organizations to compile with "warnings are errors" turned on. Therefore, any time we introduce a new warning for a pattern that is often actually safe and frequently used, we are effectively causing an enormous breaking change. A vaccine which kills more healthy people than the disease would have is probably not a good bet. (\*\*)

This is not to say that a warning is a bad idea, but that it is not the obvious slam dunk good idea that it initially appears to be.

A number of people suggested that the problem was in the training of the developers, not in the design of the language. I disagree. Obviously modern languages are complex tools that require training to use, but we are working hard to make a language where people's natural intuitions about how things work lead them to write correct code. I have myself made this error a number of times, usually in the form of writing code like the code above, and then refactoring it in such a manner that suddenly some part of it escapes the loop and the bug is introduced. It is very easy to make this mistake, even for experienced developers who thoroughly understand closure semantics. That's a flaw in the design of the language.

And finally, a number of people made suggestions of the form "make it a warning in C\# 4, and an error in C\# 5", or some such thing. FYI, C\# 4 is DONE. We are only making a few last-minute "user is electrocuted"-grade bug fixes, mostly based on your excellent feedback from the betas. (If you have bug reports from the beta, please keep sending them, but odds are good they won't get fixed for the initial release.) We are certainly not capable of introducing any sort of major design change or new feature at this point. And we try to not introduce semantic changes or new features in service packs. We're going to have to live with this problem for at least another cycle, unfortunately.

\*\*\*\*\*\*\*\*

(\*) Mr. Smiley Face indicates that Eric is indulging in humourous japery.

(\*\*) I wish to emphasize that I am 100% in favour of vaccinations for deadly infectious diseases, *even vaccines that are potentially dangerous*. The number of people made ill or killed by the smallpox vaccine was tiny compared to the number of people who did not contract this deadly, contagious (and now effectively extinct) disease as a result of mass vaccination. I am a strong supporter of vaccine research. I'm just making an analogy here people.

(This is part two of a two-part series on the loop-variable-closure problem. [Part one is here](http://blogs.msdn.com/b/ericlippert/archive/2009/11/12/closing-over-the-loop-variable-considered-harmful.aspx).)

