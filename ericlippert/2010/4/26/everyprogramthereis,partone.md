# Every Program There Is, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/26/2010 7:07:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/22/every-tree-there-is.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/29/every-program-there-is-part-two.aspx).\]

We can now enumerate every binary tree and every arbitrary tree of a given size, and therefore we can enumerate all of them, period: enumerate all the trees of size one, then all the trees of size two, and so on. This certainly is interesting for generating test cases for algorithms that consume trees. However, I also write algorithms that take *programs;* of course a compiler is chock full of algorithms that manipulate parts of programs. Can we use these same techniques to automatically generate test cases for compilers?

It seems plausible that we ought to be able to do so. I chose my notation for arbitrary trees as sequences of nested parentheses for a reason. If we take one of those strings, say {{}{{}}}, and we do a search-and-replace replacing the first { with class A {, and each subsequent one with class B { and so on then we get (with whitespace added)

 

class A  
{  
  class B { }  
  class C { class D { } }  
}

In short, we can use our binary tree generator to generate all possible nestings of classes just by making minor modifications to the code which prints out the string associated with the tree.

We can go farther than this though. I want to see if we can come up with some more general mechanism for generating all members of a particular language.

To do this I’m going to start today by describing an incredibly important tool in the design of programming languages, the Context Free Grammar, or henceforth, CFG.

It’s probably easiest to simply start with an example. Here’s a CFG for some simple declarative English sentences:

 

SENTENCE:  SUBJECT VERB OBJECT  
SUBJECT:   ARTICLE ADJECTIVE NOUN  
OBJECT:    ARTICLE ADJECTIVE NOUN  
ARTICLE:   a | one | the  
NOUN:      dog | man  
ADJECTIVE: fat | short  
VERB:      chases | bites

The idea is that any time you see one of the things on the left, you can substitute one of the things on the right. For example, we could start with SENTENCE. Only one legal substitution there: SUBJECT VERB OBJECT. Then we make a substitution for SUBJECT; again, there is only one legal substitution, and now we have ARTICLE ADJECTIVE NOUN VERB OBJECT. Then we make a substitution for ARTICLE. Now we have three choices; let’s say the. So we have the ADJECTIVE NOUN VERB OBJECT. Now make a choice for ADJECTIVE, say the fat NOUN VERB OBJECT. Keep on going until you cannot make any more substitutions, and you end up with something like the fat man bites the short dog.

To formalize this a bit: the words in ALL CAPS are called “nonterminals” because if you’ve still got one then you’re not done yet. The other words are called “terminals”; they cannot be further substituted. The grammar formally consists of (1) a set of distinct terminals and nonterminals, (2) a set of rules mapping each nonterminal onto a set of legal substitutions, and (3) which nonterminal is the one you start with - SENTENCE is the start symbol in our example. Rather than calling it out we'll usually just list the start symbol first.

The usual problem faced by compiler writers is the opposite of the process we just went through. That is, given a file containing only terminals (the short dog chases the fat man), go backwards: figure out exactly which rules were used to produce that statement, all the way back to the start, or, if you cannot, give a sensible error message describing why not. This is called “parsing the code”. Unsurprisingly, this is quite a difficult process for certain grammars; the grammar I’ve laid out here is relatively easy to analyze. Fortunately, in this series of articles I won’t be talking about the messy details of building parsers; we want to use CFGs to *generate* legal programs, not to *parse* them, so we’ll concentrate on that.

Where CFGs get interesting is when they become recursive. Consider this simple grammar with only two rules, two nonterminals and two terminals:

 

S: { X } | { }  
X: X X | { X } | { }

The start is S. Let's make a substitution { X }. We have three possible substitutions for X, let’s pick the first, so { X X }. (Notice that unlike in our previous grammar, we can make arbitrarily long statements\! In the previous grammar no matter what you do you end up with exactly seven terminals. Here we can end up with any even number of terminals.) Continuing on, lets go to { { } X }, and then to { { } { X } } and then { { } { { } } }, and we’re done.

Apparently we’ve come up with a grammar for all the possible arbitrary trees; we’re on the right track. If we could find a way to enumerate all members of this grammar we’d be in good shape.

**Next time:** addition causes ambiguity.

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/22/every-tree-there-is.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/29/every-program-there-is-part-two.aspx).\]

