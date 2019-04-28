# Regular Expressions From Scratch, Part Nine: A Dream of a Machine

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/15/2005 10:28:00 AM

-----

I want to come up with the *simplest possible device* that can identify whether a given string is a member of a given regular language. We need some kind of computer, but to make it easy to analyze, I want to put as many restrictions upon it as possible. For example, I want there to be very limited memory storage, I want to process only one character at a time, and so on.

Definition 14: A **deterministic finite automaton** (DFA) is an idealized machine. It has an input **tape** with a finite string written down on it in a given alphabet. The tape is exactly as long as the string. The DFA has a finite number of distinct **states** that it can be in. A DFA can be in only one state at a time, called the **current state**. The DFA always starts in a particular state called the **start state**. The DFA reads the string from left to right, one symbol at a time. There is no backtracking allowed. Each new symbol on the input tape may cause the machine to change the current state, but this choice can only be based on the current symbol and current state, and the rules for how that choice is made must be specified in advance and cannot change. Every state is either an **acceptable state** or an **unacceptable state**. When the machine is done reading the string, if it is in an acceptable state then the machine **accepts** the string, otherwise it **rejects** it.

I think you'll agree that this is a very limiting set of restrictions to place upon a computer.

We'll write down DFAs like this example:

Let M be a DFA such that:  
Alphabet: S = {a, b}  
States: K = {0, 1}  
Start: s = 0  
Acceptable: F = {1}

Every DFA has rules for determining what the state transitions are, one rule for every possible combination of states and symbols. For example, M might have rules:

current state

current input

new state

0

a

1

0

b

1

1

a

0

1

b

1

Note that there MUST be a rule for every combination of K and S. Since it will get cumbersome to write out these tables constantly I'm going to declare a new notation:

(0, a) → 1  
(0, b) → 1  
(1, a) → 0  
(1, b) → 1  

The arrow is read as "yields".

Consider the action of M on a tape containing aab. The machine uses these rules in this order:

(0, a) → 1  
(1, a) → 0  
(0, b) → 1  

The string is accepted because we've run out of string and we're in an acceptable state.

Since any set of strings is a language, the set of strings which a machine M accepts forms a language. We can construct a finite sequence of sets (alphabet, states, transition rules, etc) that exactly characterizes M, so we can build a function L which maps M onto the language which M accepts. Call the language L(M).

Convince yourself that in the case above, L(M) = { (a(aa)\*) ∪ (((a ∪ b)\*b) (aa)\*) } - that is, the regular language where every string is either an odd number of as or any string ending in a b followed by an even number of as.

This idea of listing the rules used by the machine is pretty good, but it requires us to mentally keep track of how far along the input tape the machine is. Also, there's some redundancy there, since obviously the state of the next line is going to be the same as the new state of the current line. I'm therefore going to change the notation I use to describe how a machine is working. For the example above we'd say in our new notation

(0, aab) → (1, ab) → (0, b) → (1, e)

Clearly this is the very definition of a mechanical process. It's going to get tedious to write out all of the steps in more complex machines. Let me declare a new "super arrow", which means "yields eventually".

(0, aab) ⇒ (1, e)

I'll use the super arrow to mean "yields in zero or more steps". Maybe one, maybe a thousand, but some finite number of steps.

What's the point of a DFA? It's pretty much the simplest thing we can possibly call a computer. It's got input and output and storage, but is very limited. The input, sure, it can be as long or as short as you want. But the output and the storage consists of a single "register" which can only hold one of a finite number of states. In our example today, we've got only a *single bit* of storage and four rules, and we can already accept a fairly complex language.

If we can reason about the limits of a DFA then we can determine whether a machine with finite storage is buff enough to recognize any interesting languages. Today's machine recognizes a regular language, and that's a start.

Next time, we'll add a little magic to a DFA.

