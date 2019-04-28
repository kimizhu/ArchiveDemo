# Regular Expressions From Scratch, Part Seven: Listing All Members Of A Language In Order

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/8/2005 10:00:00 AM

-----

Regular languages are by definition those languages which can be described by a regular expression. It should be clear from the definition that the union of any finite number of regular languages is a regular language, the concatenation of any finite number of regular languages is a regular language, and the Kleene Closure of any regular language is a regular language.

That's rather a lot of languages. Are *all* languages regular languages?

We should have a strong intuition that regular languages are a very limited subset of all languages. It seems like it would be hard to come up with a finite regular expression that, say, matches the language {w is in 1\* such that there are a prime number of 1s in w}.

But we can do better than intuition. There are many interesting ways to prove that there exists at least one non-regular language. Over the next couple entries we'll find a non-regular language by using a clever trick. The first thing we'll need to do is enumerate every member of a language.

It doesn't really matter how we enumerate the language, as long as we guarantee that we eventually hit every member of the language. Here's one way to do such an enumeration.

Consider an alphabet, say S = {a, b}. We can enumerate all the strings of S\* first by length and then by alphabetical order. Of course, we'll need to define an order for our alphabet, but since alphabets are by definition finite sets, we can choose any old order. In general, for alphabets consisting of Roman alphabet characters and numbers we'll use the standard alphabetical ordering we're all used to. We'll start with the one zero-length string, then the two one-symbol strings, then the four two-symbol strings, and so on:

e, a, b, aa, ab, ba, bb, aaa, aab, aba, abb, …

(A commenter correctly pointed out that there are languages which cannot be easily enumerated in this way, but my coming argument does not rely upon an alphabetical ordering. All we need is some schema for enumerating a language which guarantees that we eventually get all of them.)

Consider R, the regular expression language of S. We can enumerate it too, first by length and then by alphabetical order. Let's say that alphabetical order of R is \* ( ) a b Ø ∪ just to pick an arbitrary ordering for the symbols. We can then enumerate R in the same way like this:

a, b, Ø, a\*, b\*, Ø\*, a\*\*, b\*\*, Ø\*\*, (aa), (ab), (aØ), (ba), (bb), (bØ), (Øa), (Øb), (ØØ), …

Of course we are leaving out any strings which are not in R, such as \* or ))(.

Next time we'll use the fact that we can make both these lists to show that a non-regular language exists.

