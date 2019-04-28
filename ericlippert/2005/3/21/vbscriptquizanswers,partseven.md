# VBScript Quiz Answers, Part Seven

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/21/2005 4:58:00 PM

-----

Before I get into today's entry, a brief editorial comment: Pestilence is a frickin' *pain in the rear*. Famine was no problem at all, Rodney didn't touch me once, and Master Kaen fell to my powerful fists, but two Amulets of Life Saving, one Wand of Death and four potions of full healing were insufficient to keep Pestilence at bay. That's the *second* time Pestilence has ruined a perfectly good ascension. I need to figure out how to take that guy out. (If you have no idea what I'm talking about, it's probably for the best.) Onwards\! 7) What does this silly program do? On Error Resume Next  
For i = 1 To 1/0  
  Print "x" & i & "x"  
next  
Print "Done"  
  
(a) prints xx, Done  
(b) prints x1x, Done  
(c) prints Done  
(d) Goes into a loop, producing x1x, x2x, etc. It does (a). This illustrates something interesting about the implementation of For-Next loops. It looks like what *ought* to happen here is first 1 is assigned to the loop variable, then the limit produces an error. We'd resume into the loop, but then what happens when we get to the bottom and try to compare to the limit? That's not what happens. In fact, all three parameters -- the beginning, ending and step -- are calculated *before* the assignment to the loop variable. When the calculation of the limit fails the assignment never happens, so the loop variable keeps whatever value it had before. The loop variable was never initialized, so it's Empty. Control resumes inside the loop. When we hit the Next statement it detects that the For loop was never initialized correctly, raises the seldom-seen "loop not initialized" error, and resumes outside the loop. There is no end to the shens you can pull with On Error Resume Next. It makes programs very hard to reason about.

