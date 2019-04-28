# Regular Expressions From Scratch, Part Four: The Kleene Closure of a Language

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/28/2005 10:00:00 AM

-----

Languages are sets, so we can take any two languages (over the same alphabet) and take their union to form a new language. Just as a reminder:

**Definition 7**: The **union** of two sets L and K is the set with all the members found in *either*, and is written L ∪ K.

We're going to take the Kleene Star one level further and say that it applies to languages too.

**Definition 8**: The **Kleene Closure of a language** L is L\* = { e } ∪ L ∪ LL ∪ LLL …

Got it? L\* is the language consisting of set of strings, where those strings are any number of strings in L concatenated together.

I want to extend our notion of concatenation in one more way. When I write a string or string variable next to a language, I mean "concatenate the language onto the end of the language that consists of only this string". So when I say abLa I mean {ab}L{a} Again, you can infer the braces from the context.

Let's do an example of using \* and string concatenation of languages, just so its clear.

S = {0, 1}  
L = {011\*}  
1L\* = {1, 101, 1011, 101101, 1011011, …}

Or, in English, this is the set of strings w in S\* such that w:

  - starts with a single 1,
  - has no more than one 0 in a row, and
  - ends with 1

We need one more thing to make this notation sufficiently terse. You probably noticed that the Kleene Star above only applied to the thing immediately to the star's left, whether the language or the symbol. Suppose we've got four languages: H, J, K and L. We can form two new languages through concatenation: G = HJ and F = KL. And then we can form a seventh language, D = G\*F\*. But we can't write D = HJ\*KL\*, because that's a different language. What we mean is D = (HJ)\*(KL)\*.

Therefore I'm declaring that we can use parenthesis in the normal way we're all used to for order of operations. We will figure out from context whether parens mean "this is a finite sequence" or "this is a grouping of concatenations".

Remember, we're looking for a way to concisely characterize languages, and clearly we're onto something here. We now have **union**, **grouping**, **concatenation** and **Kleene closure**. In other words, we have enough tools to define regular expressions, which we'll do next time. Stay tuned\!

