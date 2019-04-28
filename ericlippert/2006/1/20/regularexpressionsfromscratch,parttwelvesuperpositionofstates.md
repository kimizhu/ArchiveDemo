# Regular Expressions From Scratch, Part Twelve: Superposition of States

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/20/2006 2:47:00 PM

-----

Happy New Year everyone. Over the break I had a wonderful time reconnecting with my friends and family. And of course I came back to a huge pile of work\! We're going through the flaws that were discovered in C\# 2.0 too late in the cycle to risk fixing, and some of them illustrate interesting corner cases in the language. But those will have to wait, as we've still got a lot of ground to cover before I'm done this crazy long series on regular expressions.

To summarize the story so far: we've defined alphabets and languages over those alphabets. We've shown how to create a special "regular expression" alphabet and language for any given alphabet. We've also come up with a rule that associates a language with a regular expression. This language is the set of strings which "matches" the regular expression.

We've determined that there are deterministic finite state computers which consume one character of a string at a time that can "recognize" strings from some regular languages. We've also determined that there are "nondeterministic" finite automata that somehow "magically" figure out which rules to apply at any time to match a string.

We've also shown that any NFA that has rules that act on more than one character can be turned into an equivalent NFA which only has single-character or no-character transitions.

What we would like to do is show the following three facts: first, that every NFA is equivalent to a DFA. Second, that every DFA/NFA recognizes a regular language. And third, that every regular language has a DFA/NFA that recognizes it.

We've still got a ways to go to show that first result. Rather than prove the general result, which will be tedious, I'll just go through an example and hope that it is illustrative enough to convince you that we could give this treatment to any NFA.

Let M be an NFA with alphabet {a, b}, states {1, 2, 3, 4}, start state is 1, and the set of acceptable states is {4}. We'll assume that we've already eliminated all the multi-character rules and have a set of rules as follows:

(1, e) → 2  
(1, b) → 3  
(2, a) → 1  
(2, e) → 3  
(2, a) → 4  
(3, b) → 4

This NFA accepts the language a\*( bb ∪ b ∪ a ), and as you can see, we've got lots of empty and ambiguous rules.

Here's the trick: given an NFA with 4 states like this one, we can find a DFA which is equivalent to this guy that has 2<sup>4</sup> = 16 states or fewer. We can think of the NFA as at any time living in a "superstate" that consists of all of the states that it COULD be in right now.

Let me try to explain that a little better. We start in state 1, right? But since (1, e) → 2 and (2, e) → 3, we could also be starting in states 2 or 3 before we process any input. Let's create a new automaton with a start state that represents the concept "right now M could be in states 1, 2, or 3." We'll call that state 123x, which is one big symbol, not a string of four symbols. In our new automaton, the start state is 123x.

We're trying to write a DFA here, so there needs to be a rule for every input and every state:

(123x, a) → ?  
(123x, b) → ?

Look at the original automaton. When we were in states 1, 2 or 3, what were the possible state transitions for a? The only ones were (2, a) → 1 and (2, a) → 4. But we're not finished\! Again, we need to consider what could happen from states 1 and 4 if we processed the "empty string" transition rules. Since (1, e) → 2 and (2, e) → 3, we could end up in state 1, 2, 3 or 4. Let's create a new state called 1234 for our DFA and finish off that rule:

(123x, a) → 1234

Now do the same analysis for (123x, b) and we'll see that in the original automaton, the only possible resulting states starting in 1, 2 or 3, and processing a b are 3 and 4. Add another new state:

(123x, b) → xx34

Are we done? No. We've added two more states, 1234 and xx34, so we need to figure out the state transitions for them too, which we do by the same process. We discover that

(1234, a) → 1234  
(1234, b) → xx34  
(xx34, a) → ?  
(xx34, b) → xxx4

Uh oh. There are no state transitions in the original NFA that start in states 3 or 4 and take an a. Remember that in that case we assume that the NFA goes into a special "reject" state. Let's call the reject state xxxx.

We now have two more new states to work out the transitions for. If we're in the reject state, we stay in the reject state, and we see from the original NFA that there are no transitions out of state 4, so we can round out our list with:

(xx34, a) → xxxx  
(xxxx, a) → xxxx  
(xxxx, b) → xxxx  
(xxx4, a) → xxxx  
(xxx4, b) → xxxx

Since 4 was the acceptable state in the original NFA, any "superstate" that contains 4 must be an acceptable state. So our new DFA is a machine M<sub>2</sub> with alphabet {a}, b}, states {1234, 123x, xx34, xxx4, xxxx}, start state is 123x, and the set of acceptable states is {1234, xx34, xxx4}. The rules are

(1234, a) → 1234  
(1234, b) → xx34  
(123x, a) → 1234  
(123x, b) → xx34  
(xx34, a) → xxxx  
(xx34, b) → xxx4  
(xxx4, a) → xxxx  
(xxx4, b) → xxxx  
(xxxx, a) → xxxx  
(xxxx, b) → xxxx

This DFA accepts the language a\*( bb ∪ b ∪ a ), so we've found an equivalent DFA to our NFA. We've turned an NFA with four states and five rules into an equivalent DFA with five states and ten rules. We did well -- you can find NFAs that require 2<sup>n</sup> new states, where n is the number of NFA states. An NFA with a thousand states could require 2<sup>1000</sup> states to represent as a DFA\! 2<sup>1000</sup> is a finite, albeit rather large number. But it's not *that* big. Clearly we could build a 2<sup>1000</sup> state machine with only 1000 bits to store the state information. It's the *state transition rules* that are the pain to work out\!

Using the techniques from this and the previous entry we can turn any NFA into an equivalent DFA, so NFAs are really not magic at all. They're just a convenient way to talk about DFAs.

Now, obviously what I've done here isn't a proof; one example does not a proof make. But the proof is both tedious and complicated, so I'm going to skip it. The key result here is that we can stop saying "deterministic finite automaton" and "nondeterministic finite automaton" and just start saying "finite automaton", because they're essentially the same thing.

Now that we know that we can use NFAs with impunity, we can try to answer questions such as:

Is every regular language recognizable by a finite automaton?

Are there any FAs that recognize irregular languages?

Tune in next time to find out\!

