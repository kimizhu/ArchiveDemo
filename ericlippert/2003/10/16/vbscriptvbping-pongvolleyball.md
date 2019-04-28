# VBScript : VB :: ping-pong : volleyball

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/16/2003 5:08:00 PM

-----

It's my girlfriend Leah's 30th birthday today\! Happy birthday Leah\!

Leah is a tester in the Mobility division here at Microsoft, where she works on the software that moves email between servers and cellphones. Right now she and her fellow testers are boning up on their C\# skills so that they can write better testing automation scripts. Surprisingly, a lot of the sync software testing is still done "by hand", rather than writing a program to run the same test suites over and over again every day.

The other day Leah pointed out to me something that I've been intending to write about -- that often it really doesn't matter what language you learn.  For many programmers, the language is just the thing that stands between you and the object model.  People writing application automation need to be able to create instances of classes, call methods, listen to events, run loops, and dump strings to log files.  Learning the syntax for those operations in JScript, VBScript, VB, C\#, C++, perl (shudder), Python, etc, is trivial.  The complexity lies in understanding the object model that you're manipulating so that you can use it well to test the product.

The same goes for the vast majority of scripting applications.  People ask me *"Eric, I'm a newbie programmer -- should I learn VBScript or JScript?"*  I tell them that script is glue and that what matters is that you glue the right things together, not that you pick the right glue. 

That's not to say that there aren't important differences between languages.  As I mentioned the other day, some languages are designed to support programming in the large, some are designed to facilitate rapid development of small, throwaway programs. Some are for writing device drivers, some are for research, some are for science, some are for artificial intelligence, some are for games.  If you have complex structures that you wish to model, it’s a good idea to pick a language that models them well.  Prototype classes (JScript) are quite different from inheritance classes (C\#), which in turn are different from simple record classes (VBScript).  But my point is that by the time you're ready to write programs that require these more advanced features, you'll be able to pick up new languages quickly anyway. 

And this is also not to say that automation testing is *just* glue code. I've had many long conversations with the testers on my team on the subject of writing automation tools.  When you move from the low level automation code (call this method, did we get the expected response?) to the higher-level (run tests 10, 13 and 15 against the VB and C\# code generators for the Word and Excel object models on 32 and 64 bit machines but omit the 64 bit flavours of test 15, we already know that its broken) then you start to run into much deeper problems that may require their own object models to represent.  Or even their own languages\!  One of the testers I work with is kicking around some ideas for a "test run definition language" that would cogently express the kinds of massive test matrices that our testers struggle with.

But these are not newbie programmer problems.  If you're just getting into this whole scripting thing, pick a language and use it to learn the object model inside-out.  Once you know how the OM works, doing the same thing in a different language should be pretty straightforward. 

It's kind of like table tennis.  If you know the rules of table tennis, learning the rules of real tennis is pretty easy -- it's table tennis, just with a larger board.  And you stand on the board, and the ball is bigger, as are the racquets.  But, as George Carlin said, it's basically the same game.  And if you know tennis, volleyball is pretty easy -- it's just tennis with nine people on a side and you can hit the ball three times and it can't hit the ground.  And there are no racquets, and the ball is bigger and the net is higher, and you play to 21 points. But it's basically the same game. 

OK, maybe that's not such a good analogy.  But you take my point, I'm sure.  Don’t stress about choice of language, but learn the object model cold.  The question shouldn't be "what language should I learn" but rather "what object framework solves my problem, and what languages are designed to efficiently use that framework?"

