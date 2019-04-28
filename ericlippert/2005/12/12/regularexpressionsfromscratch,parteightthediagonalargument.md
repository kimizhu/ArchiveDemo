# Regular Expressions From Scratch, Part Eight: The Diagonal Argument

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/12/2005 10:00:00 AM

-----

As we know, each regular expression is associated with a language by our function L. We also determined last time that we could list all members of S\* and R in order first by length and then by alphabetical order. Here's a weird question: is the nth string in S\*'s alphabetical ordering a member of the language associated with the nth alphabetical regular expression?

Let's make a table and see if we can discern any pattern.

s

r

s in L(r)?

e

a

no

a

b

no

b

Ø

no

aa

a\*

yes

ab

b\*

no

ba

Ø\*

no

bb

a\*\*

no

aaa

b\*\*

no

aab

Ø\*\*

no

aba

(aa)

no

We could continue this table forever, of course, and it seems that sometimes, pretty much at random, we're going to get a match. Matches might not be very often, but they will happen from time to time. As it turns out, it doesn't really matter whether we can come up with some pattern here. All that matters is that we can answer the question definitively for every possible entry.

Now define the language D such that D = { w in S\* such that w has "no" in the column above }

D = {e, a, b, ab, ba, bb, aaa, aab, aba, abb, … }

This is a weird but perfectly well-defined property. Given *any* string we can determine very quickly whether it is a member of D or not. Just figure out where on the table it is, compute the nth regular expression, and see if it is a member of that regular language. If it is not, then it is in D.

Is there any regular expression that specifies the language D?

Which one would it be? We've got a *complete alphabetical list* of regular expressions, so let's just go down the list. By our definition of D, clearly it can't be *any* of them. If the third column is "yes" then the nth regexp matches a string not in D. If it is "no" then the nth regexp fails to match a string in D. Therefore there is no such regular expression that matches everything in D. Therefore D is not a regular language.

By a similar argument we can show that for every nonempty alphabet there exists a language which is not regular.

That's a pretty unlikely example of an irregular language though. Later we'll see that many perfectly normal languages are not regular, including, ironically enough, the regular expression language itself.

Remember that the reason why we came up with regular expressions in the first place is because we wanted a concise way to characterize languages using short, simple expressions. We of course could come up with other mechanical means to cleverly map between strings in one alphabet and languages in another, but ultimately it wouldn't do us much good if our aim is to capture all possible languages. The diagonal argument above doesn't depend upon any special features of regular expressions; it applies to *any* function that maps strings onto languages.

**No language description system which maps finite strings onto languages can describe every language.**

This is a stunning result; there are languages which we cannot characterize in any finite number of symbols, no matter how clever we are with our symbolic manipulations\!

It's simply a fundamental fact that the definition of "language" we've chosen - an arbitrary set of finite strings - affords an immense number of possible results, more immense than the number of strings in any description language you'd care to name. Some - infinitely many - will be indefinable.

You might have noticed a bit of a hand-wave in today's entry: I've made the claim that we can easily take a regular expression and determine if a string is in its regular language. Over the next few entries we'll justify that claim by building some simple abstract machines that can make this determination.

