# No backtracking, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/4/2010 9:56:00 AM

-----

A number of the articles I’ve published over the years involve “backtracking” algorithms; most recently my series on how to solve Sudoku puzzles (and more generally, all graph colouring problems) by backtracking. The idea of a backtracking algorithm is really simple and powerful: when faced with a choice, try every possibility. If all of them go wrong, backtrack to the previous choice and try a different possibility. Backtracking algorithms are essentially a depth-first search of the space of possible solutions.

But what if the solution space is large? There are more ways to fill in nine digits in 81 places than there are protons in the entire Universe; we can’t check them all. The reason that backtracking works so rapidly for Sudoku puzzles is because typically every guess eliminates many of the branches of the solution tree. If you guess that the first square is 1 then you do not need to explore the possibility that the second square is 1; you can just prune that branch and never explore it. Since that branch itself is Vast, that’s a good strategy\! (There are pathological cases for this algorithm, but in practice it is fast.)

Backtracking algorithms work well for many situations, but we have consciously eschewed them in the design of C\#. (With a few exceptions).

For example, there is no backtracking that ever crosses “phases” of compilation in C\#.(\*) When we are presented with some source code the first thing we do is “lex” it; break it up into “tokens”. This is akin to finding the words in a sentence by looking at the spacing and punctuation. Then we do a grammatical analysis of the token stream, and then we do a semantic analysis of the grammatical analysis. When one of those phases finds an error, we never backtrack to a previous phases and ask for more analysis. Suppose for example you have this silly code:

 

x = j+++++1;

That is *lexically* correct C\# code. The lexer is a greedy lexer; it tries to make the largest token it can at any given time. So it tokenizes this as:

 

x , = , j , ++ , ++ , + , 1 , ;

Now the grammar analyzer takes that token stream and says “this is an expression involving two identifiers, one constant and four operators; how should I parenthesize that?” The grammar analyzer determines using the rules of precedence and associativity that this means

 

x=(((j++)++)+1);

So this means “evaluate x, then increment j, then increment whatever the previous result was, then add 1 to the previous result, then assign the previous result to x, done”

Then the semantic analyzer checks that to make sure that all those operations make sense. They do not, because the result of j++ is a value, not a variable, but the second ++ requires a variable. The semantic analyzer gives an error, and the program does not compile.

If we had a backtracking compiler, the semantic analyzer could tell the parser “no, you did something wrong, give me a different parse tree if there is one”.

Suppose the parser does that. It could say well, maybe that last + was not a binary operator but rather the unary operator + applied to 1, oh, but if we do that then we have two expressions next to each other with no intervening operator. Same thing if the second ++ applies to the +1 instead of the j++. So no, there is no other parse of this. So push back on the lexer and ask it to backtrack.

The lexer could then say, oh, sure, if you don’t like that then maybe my guess that the second ++ was ++ was wrong. Try this one:

 

x , = , j , ++ , + , ++ , 1 , ;

and then the parser says OK, if that's where the token breaks are then the grammatical analysis is:

 

x=((j++)+(++1));

and the semantic analyzer determines that’s wrong too, this time because you can’t have a ++ on a constant. So the semantic analyzer tells the parser, no, that’s not it either, and the parser tells the lexer, no, that’s not it either. And the lexer says “well, maybe that wasn’t a ++ at all the second time, maybe that should be

 

x , = , j , ++ , +, + , + , 1 , ;

And now the parser says “oh, I see what that is, it is two unary plus operators applied to 1 and an addition operator:

 

x=((j++)+(+(+1)));

and the semantic analyzer says “finally\! something I can work with.”

That’s not what we do. This is a silly example, but perhaps you can see that there are two major problems with this strategy.

First, though most of the time lexing is unambiguous, in some pathological cases there are potentially enormously many ways that simple programs could be lexed. Just determining where the breaks could go between five plus signs gives eight possibilities; it grows exponentially from there. Since most of the time programs lex unambiguously, and when they do lex ambiguously, the space to explore could be really big, it’s better to always lex greedily and not backtrack ever. If a program fails grammatical analysis because of an unintended lexical structure then the best thing to do is tell the user and they’ll fix it.

The second major problem is that when backtracking, how do you know which backtracking is better? Of those eight possibilities, two give program fragments that pass semantic analysis: the one we just found, and x=(j+(+(+(+(+1))))); Is it at all clear which of the two possibilities is the one the user meant when they wrote this ridiculous statement? Remember, we’re not trying to solve a Sudoku puzzle here, where any given combination of numbers is as meaningless as the next; we are attempting to output IL that has the same meaning as the meaning intended by the user\! One of these possibilities mutates j and the other does not, so it makes a difference. C\# is not a “guess what you meant” language, it’s a “do what you said” language(\*\*). If what you said was ambiguous, we should tell you about it.

Next time we’ll return to this point by looking at how the name resolution algorithm also does not backtrack.

(\*) Unlike JScript, interestingly enough. The ECMAScript specification notes that there are lexical ambiguities involving the / character; it could be introducing a comment as // or as /\*, or closing a comment as \*/, or could be a normal division operator, or a /= operator, or opening or closing a regular expression. Some of those overlap; The spec actually says that the correct lex is the one that produces a grammatically sensible program\!

(\*\*) Again, unlike JScript.

