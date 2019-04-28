# Regular Expressions From Scratch, Part Ten: Magic\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/19/2005 10:00:00 AM

-----

Let's recap the story so far.

Starting only with basic set theory, sequences, symbols and numbers, we've defined alphabets, languages, regular expressions, and the mapping between regular expressions and regular languages. We've also defined deterministic finite automata as machines that take in strings one character at a time, change their internal state to one of a finite set according to strict rules, and end up in either an accept or reject state as output.

Where we're going with this is towards unification of these two concepts. We want to show that for every DFA there's a regexp, and vice versa. But to get there is going to take some magic.

The trouble with DFAs is that they're kind of a pain to specify. You have an alphabet with, say, a hundred symbols, and you have, say, fifty states. You need to come up with a state transition rule for 100 x 50 = 5000 possible combinations. Since most of those will likely be transitions to rejection states anyway, that's majorly boring.

I'm hereby declaring that we have **nondeterministic finite automata**. An NFA works just like a DFA, except that the "rules" for determining the state transitions can be **ambiguous and weird**. We'll write an NFA like this:

Let M<sub>1</sub> be an NFA such that:

Alphabet: S = {a, b}  
States: K = {0, 1, 2}  
Start: s = 0  
Acceptable: F = {0}

Rules:

(0,a) → 1  
(1,b) → 2  
(2,a) → 0  
(2,e) → 0

Notice that we're not specifying the six rules we ought to be, and one of those rules is a transition on an "empty input"\! This last rule means that if we are in state 2, we can go to state 0 "for free", without consuming any characters.

We say that M<sub>1</sub> accepts a string if there is *any* way to make it eventually yield an acceptable state with no more input. For example, suppose we start with ababa:

(0, ababa) → (1, baba) → (2, aba) → (0, aba) → (1, ba) → (2, a) → (0, e)

is one path to an acceptable state. Even though

(0, ababa) → (1, baba) → (2, aba) → (0, ba) gets us stuck in a state that we can go no further in, that's OK. NFAs are magic. If there exists *any* path such that (0, ababa) ⶒ (0, e) then the NFA will find it and end in an accepting state. **The fact that some nonsensical or rejecting paths exist doesn't matter if an accepting path exists.** Only if every possible path either ends in a rejecting state, or leaves us unable to find any applicable rule, does the NFA reject the string.

In fact, this machine accepts the regular language (ab∪aba)\*

(Recall that I am being sloppy about parenthesizing. This isn't a "real" regular expression but you get the point that when I say that I mean L(((ab)∪((ab)a))\*) I'm sure.)

We've got lots of magic already, so let's add more. We'll say that a rule can consume any number of matching symbols that it wants.

Let M<sub>2</sub> be an NFA such that:

Alphabet: S = {a, b}  
States: K = {0}  
Start: s = 0  
Acceptable: F = {0}

Rules:

(0,ab) → 0  
(0,aba) → 0

This machine clearly accepts (ab∪aba)\* except for the fact that it is now totally *unclear* what we mean by "accepts" vs. "rejects" in a machine with only one state\! How does it reject anything?

Let's assume that NFAs have a magic implicit "crash" state which means "For this input I was unable to find any path that did not end in a state where I lacked a rule telling me what to do next." The crash state is always a rejecting state.

Clearly M<sub>1</sub> and M<sub>2</sub> accept the same language. Let's make up a word for that relationship.

**Definition 15:** Two machines are **equivalent** if they both accept exactly the same language.

Next time: Can we in fact build a device which does magic?

