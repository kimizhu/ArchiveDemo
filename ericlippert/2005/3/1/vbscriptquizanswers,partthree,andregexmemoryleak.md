# VBScript Quiz Answers, Part Three, and RegEx Memory Leak

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/1/2005 10:21:00 AM

-----

Co-Winner Nicholas Allen's prize is en route to his home now. Steven Bone, I need your mailing address still. Before I give the answer to \#3, a quick note. NTBUGTRAQ is reporting a memory leak in the VBScript regular expression parsing code. I took a quick glance at the sources, and sure enough, there is a potentially large leak when calling Execute on a regular expression **with more than ten subexpressions** in **global mode**. (The leak is in code shared by VBScript and JScript, so odds are very good that JScript has the leak as well. I have not verified that.) The flawed code which caused the leak was checked into the VBScript sources on the tenth of November, 1999. If you're affected by the leak -- for instance, if you're running an ASP page which does lots of complex global regular expressions -- then I do NOT recommend that you roll back to a version of the script engine from before 1999\! Rather, see if you can simplify your regular expression to fewer than ten subexpressions, or do not use global mode. I'm working with the sustaining engineering team, but no word yet on when a fix will be made available. I apologize for the mistake and any inconvenience thereby caused. And thanks to my old VBA buddy Ilan for bringing this to my attention. Now, onward\! \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 3) Which of the following are syntactically legal? Why? (a) x.y  
(b) x. y  
(c) x . y  
(d) x .y  
(e) x \_  
      . y (a) is obviously legal.  
(b) is not legal in VB6, but the editor removes the space for you; VBScript pretends that the space wasn't there.   
(c) is not legal -- the VB6 editor removes the second space for you, turning it into case (d). But VBScript does not, it produces an error.  
(d) is legal in this context:  With z : x .y : End With, and is essentially the same as  x z.y  So why is (e) legal then, if (c) is illegal? Because the line continuation character causes the scanner to eat all the spaces before and after the underbar.  As far as the scanner is concerned, (e) is the same as (b). Interestingly enough, this is the only situation in VBScript in which *whitespace* *between tokens* affects the *parser*. Obviously whitespace has a huge impact on the *scanner* -- 1 2 is very different from 12 -- but there is only the one situation in which it matters whether there were spaces or not between two tokens that could be lexically differentiated *with or without* spacing.

