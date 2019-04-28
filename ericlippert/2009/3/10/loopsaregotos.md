# Loops are gotos

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/10/2009 12:42:00 PM

-----

Here's an interesting question I got the other day:

> We are writing code to translate old mainframe business report generation code written in a BASIC-like language to C\#. The original language allows "goto" branching from outside of a loop to the interior of a loop, but C\# only allows branching the other way, from the interior to the exterior. How can we branch to the inside of a loop in C\#?

I can think of a number of ways to do that.

First, **don't do it**. Write your translator so that it detects such situations and brings them to the attention of a human who can analyze the meaning of the code and figure out what was meant by the author of this bad programming practice. Branching into the interior of a loop is illegal in C\# because doing so would skip all of the code that is necessary to ensure the correct behaviour of the loop. Odds are good that this pattern indicates a bug or at least some bad smelling code that should be looked at by an expert.

Second, this pattern is not legal in C\# or VB.NET, but perhaps it is legal in another useful modern language. You could write your translator to **translate to some other language** instead of C\#.

Third, if you want to do this automatically and you need to use C\#, the trick is to **change how you generate your loops**. Remember, loops are nothing more than a more pleasant way to write a "goto". Suppose your original code looks something like this pseudo-basic fragment:

10   J = 2  
20   IF FOO THEN GOTO 50  
30   FOR J = 1 TO 10  
40     PRINT J  
50     PRINT "---"  
60   NEXT  
70   PRINT "DONE"

The "obvious" way to translate this program fragment into C\# doesn't work:

     int J = 2;  
     if (FOO) goto L50;  
     for(J = 1 ; J \<= 10; J = J + 1)  
     {  
       PRINT (J);  
L50:   PRINT ("---");  
     }  
     PRINT ("Done");

because there's a branch into a block. But you can eliminate the block by recognizing that a "for" loop is just a sugar for "goto". If you translate the loop as:

     int J = 2;  
     if (FOO) goto L50;  
     J = 1;  
L30: if (\!(J \<= 10)) goto L70;  
       PRINT (j);  
L50:   PRINT ("---");  
     J = J + 1;  
     goto L30;  
L70: PRINT ("done");

and now, no problem. There's no block to branch into, so there's no problem with the goto. This code is hard to read so you probably want to detect the situation of branching into a loop and only do the "unsugared" loop when you need to.

It is difficult to do a similar process that allows arbitrary branching in a world with try-finally and other advanced control flow structures, but hopefully you do not need to. Old mainframe languages seldom have complex control flow structures like "try-finally".

