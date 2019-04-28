# Regular Expressions From Scratch, Part Eleven: Eliminating Multi-Symbol Rules

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/22/2005 10:00:00 AM

-----

The story so far: we have deterministic and nondeterministic finite automata. DFAs follow a rigid, fully specified set of rules which, on any string, yield either an accept or reject state after exactly as many moves as the string has characters. NFAs, on the other hand, are poorly specified. They somehow magically are able to choose which rules to apply given the input to find a path that yields an accept state, if one exists.

NFA magic buys us a lot, but at the high price that we can now no longer easily see a clear relationship between this magic box and a computer we could actually build. We need to get back to reality. Here is the *amazing result* that I'm going to try to justify in your minds:

***Every* NFA is equivalent to some DFA. Even better: given a description of an NFA, we can construct a description of an equivalent DFA.**

In other words, NFA magic doesn't buy us any extra power - no NFA can recognize a language that some DFA cannot also recognize.

This is excellent news, because it means that we can describe and reason about machines using short, convenient, vague nondeterministic notation, but still have confidence that we could build a fully deterministic machine that did exactly the same job.

But how the heck are we going to motivate that amazing result?

One step at a time. We're going to take some NFAs and make simple transformations that gradually turn them into different but equivalent NFAs, and eventually one of those NFAs will actually be a DFA. Rather than giving a full formal proof, which would be tedious and boring, I'm going to rely on the sketch being convincing enough that you believe me that we can take any NFA and turn it into a DFA. Maybe later in this series we'll actually write some C\# code that implements the transformation.

The magical bits that we need to remove are: rules can have multiple characters in state transitions, rules can have "empty" state transitions, rules can have ambiguous state transitions, there can be situations for which no stated rule applies, and there's a magical 'crash' state. Once we eliminate all those weirdnesses, we'll be left with a DFA.

Step one: Any NFA that has rules that have multiple-character state transitions is equivalent to some NFA that has only single-character or empty-string transitions.

We can get rid of the multiple-character transitions by adding one new state for each extra character. Recall our previous example:

Alphabet: S = {a, b}  
States: K = {0}  
Start: s = 0  
Acceptable: F = {0}

Rules:

(0,ab) → 0  
(0,aba) → 0

Suppose we added three new (non-acceptable) states, 1, 2 and 3. You agree I hope that the first rule is equivalent to these two rules:

(0,a) → 1  
(1,b) → 0

And the second rule is equivalent to these three rules:

(0,a) → 2  
(2,b) → 3  
(3,a) → 0  

We never need to add more than a finite number of new states or rules, so the new machine is always still an NFA.

Throw away the original two rules and hey presto, we have an equivalent NFA where every rule only has a zero or one-symbol string. Multi-symbol transition rules are no longer magical; we can eliminate them easily.

This example above has no empty-string transitions. Next time we'll pick another example that has empty string transitions and one-character transitions, and then show that we can turn something that looks like that into a DFA.

"Next time" will likely be in the new year, as I am heading back to my ancestral home for the next week to visit friends and relatives during this festive holiday season. I hope all of you have a fun and safe holiday; thanks for reading and we'll see you in 2006.

