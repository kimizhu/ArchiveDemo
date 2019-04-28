# VBScript Quiz Answers, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/18/2005 11:25:00 AM

-----

There were a grand total of eight entries to my VBScript quiz -- I think I made it too hard\! Congratulations to Steven Bone and Nicholas Allen, who both got all twelve right, with more-or-less correct explanations of what was happening here. Guys, send me your addresses and I'll send you both autographed books just as soon as I get my next box. Should be any day now. The answers are 1: only d is legal  
2: only c is Integer  
3: only c is illegal  
4: only a is illegal  
5: only c never prints False  
6: only d is illegal  
7: a  
8: only c is legal  
9: only a is legal  
10: only a is illegal  
11: only a is not a tautology  
12: c  
  
(Hmm, no "b"s. That wasn't intentional. Weird.) I'll spend the next few entries explaining the answers. But before I get into the details, some jargon. The standard design for a compiler is as follows: first the text of the program is broken up into **tokens** by a **lexical analyzer**, also known as a "lexer" or "scanner". This is analogous to breaking a sentence up into words, numbers, punctuation, etc. Then a **parser** organizes the tokens into larger units -- **expressions**, **statements**, **programs**. This is analogous to ensuring that a sentence is grammatical. And finally, a **code generator** generates code from the parse tree. This is analogous to translating the parsed sentence into another language. Finally, the **runtime engine** executes the generated code. The various weirdnesses I asked about are results of oddities in the design of the lexer, parser, codegen or runtime. I'll call out which is which as I go. In general, we want every VBScript program to be a legal VB6 program; VBScript is a subset of VB6. I'll also point out areas where we violate that principle. 1) Which of the following are syntactically legal VBScript statements? Why? (a) x = 10&987&654&321  
(b) x = 10&987&&654&321  
(c) x = 10&987&&654&&321  
(d) x = 10&987&&654&&&321 Only (d) is legal.  None are legal VB6 statements, so (d) is a violation of the subset principle. It's well known that you can specify hex literals in VBScript like this:  \&h123.  It's less well known that you can do the same for octal literals: \&o123.  It's even less well known that you can omit the o from the octal literal and VBScript will not complain.  The VB6 editor will automatically insert the o for you if you try to leave it out. It has always seemed bizarre to me that VBScript will accept syntaxes which the VB editor will autocorrect. The reason why the editor automatically corrects the bad syntax is **because it’s bad**\! But there are many cases where VBScript accepts the uncorrected syntax and treats it as though it were correct. In VB6 it is not legal to have the & *operator* immediately follow the left-hand operand. VB6 requires spaces around the & operator to prevent exactly this weird lexical ambiguity. VBScript does *not* require spaces around the & operator, which leads to trouble.  It's also little-known that you can specify that you want a literal to be a long integer rather than a short by appending an &.  One reader pointed out that this is a holdover from the days in VB when you could put a type decoration onto variables. foo$ was a string, for instance. The same reader also noted that VBScript is inconsistent in the semantics of the literal suffix – that’s a bug. Let’s consider (c).  Why is it illegal?  Well, look at it from the lexer’s point of view. It knows these rules:

& following a hex or octal literal is part of the literal

& followed by h, o or 0-7 is the start of a hex or octal literal

otherwise, & is a lone ampersand. Consider (c) in the context of those rules and you’ll see that it breaks up as follows: x = 10 & 987 & &654& &321 The lexer tells the parser that the program so far goes ID EQ INT AMP INT AMP INT INT EOL.  And the parser takes one look at that thing and says "this isn't a grammatical sentence."  You can't have two integers in a row without an operator between them. Similarly, you can figure out why (a) and (b) are illegal. This illustrates severalimportant points. First, the lexer is "greedy".  It tries to eat as much as it can when tokenizing each character. Second, it also tries to never look ahead more than one character to figure out what to do next. Third, the parser does not "push back" on the scanner.  The parser does not say "no, that didn't work out, see if you can find an alternative lexing that works". (The JScript parser *does* "push back" due to lexical ambiguities caused by the introduction of literal regular expressions, but that's another story\!) More next week\!

