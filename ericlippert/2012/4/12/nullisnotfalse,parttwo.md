# null is not false, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/12/2012 9:29:11 AM

-----

In [Raymond Smullyan](http://en.wikipedia.org/wiki/Raymond_Smullyan)'s delightful books about the [Island of Knights and Knaves](http://en.wikipedia.org/wiki/Knights_and_Knaves) -- where, you'll recall, knights make only true statements and knaves make only false statements -- the knights and knaves are of course clever literary devices to explore problems in deductive (\*) logic. Smullyan, to my recollection, never explores what happens when knights and knaves make statements which are disingenuous half-truths, [authorial license in pursuit of a larger truth](http://www.thisamericanlife.org/radio-archives/episode/460/retraction), or other forms of [truthiness](http://en.wikipedia.org/wiki/Truthiness). A nullable Boolean in C\# gives us, if not quite the notion of *truthiness*, at least the notion that true and false are not the only possible values of a predicate: there is also "null", whatever that means.

What does that mean? A null Boolean can mean "there is a truth state, but I just don't know what it is": for example, if you queried a database on December 1st to ask "were the sales figures for November higher than they were in October?" the answer is either true or false, but the database might not know the answer because not all the figures are in yet. The right answer in that case would be to say "null", meaning "there is an answer but I do not know what it is."

Or, a null Boolean can mean "the question has no answer at all, not even true or false". True or false: the present king of France is bald. The number of currently existing kings of France -- zero -- is equal to the number of currently existing bald kings of France, but it seems off-putting to say that a statement is "vacuously true" in this manner when we could more sensibly *deny the validity of the question*. There are certainly analogous situations in computer programming where we want to express the notion that *the query is so malformed as to not have a truth value at all*, and "null" seems like a sensible value in those cases.

Because null can mean "I don't know", almost every "lifted to nullable" operator in C\# results in null if any operand is null. The sum of 123 and null is null because of course the answer to the question "what is the sum of 123 and something I don't know" is "I don't know\!" The notable exceptions to this rule are equality, which says that two null values are equal, and the logical "and" and "or" operators, which have some very interesting behaviour. When you say x & y for nullable Booleans, the rule is not "if either is null then the result is null". Rather, the rule is "if either is false then the result is false, otherwise, if either is null then the result is null, otherwise, the result is true". And similarly for x | y -- the rule is "if either is true then the result is true, otherwise if either is null then the result is null, otherwise the result is false". These rules obey our intuition about what "and" and "or" mean logically provided that "null" means "I don't know". That is the truth value of "(something true) or (something I don't know)" is clearly true regardless of whether the thing you don't know is true or false. But if "null" means "the question has no answer at all" then the truth value of "(something true) or (something that makes no sense)" probably should be "something that makes no sense".

Things get weirder though when you start to consider the "short circuiting" operators, && and ||. As you probably know, the && and || operators on Booleans are just like the & and | operators, except that the && operator does not even bother to evaluate the right hand side if the left hand side is false, and the || operator does not evaluate the right hand side if the left hand side is true. After we've evaluated the left hand side of either operator, we \*might\* have enough information to know the final answer. We can therefore (1) save the expense of computing the other side, and (2) allow the evaluation of the right hand side to depend on a precondition established by the truth or falsity of the left hand side. The most common example of (2) is of course if (s == null || s.Length == 0) because the right hand side would have crashed and burned if evaluated when the left hand side is true.

The && and || operators are not "lifted to nullable" because doing so is problematic. The whole point of the short-circuiting operator is to avoid evaluating the right hand side, but we cannot do so and still match the behaviour of the unlifted version\! Suppose we have x && y for two nullable Boolean expressions. Let's break down all the cases:

  - x is false: We do not evaluate y, and the result is false.
  - x is true: We do evaluate y, and the result is the value of y
  - x is null: Now what do we do? We have two choices:
      - We evaluate y, violating the nice property that y is only evaluated if x is true. The result is false if y is false, null otherwise.
      - We do not evaluate y. The result must be either false or null.
          - If the result is false even though y would have evaluated to null, then we have resulted in false incorrectly.
          - If the result is null even though y would have evaluated to false, then we have resulted in null incorrectly.

In short, either we sometimes evaluate y when we shouldn't, or we sometimes return a value that does not match the value that x & y would have produced. The way out of this dilemma is to cut the feature entirely.

I said last time that I'd talk about the role of operator true and operator false in C\#, but I think I shall leave that to the next episode.

-----

(\*) Smullyan's book of *combinatory* *logic* puzzles, To Mock A Mockingbird, is equally delightful and I recommend it for anyone who wants a playful introduction to the subject.

