# VBScript Quiz Answers, Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/16/2005 10:48:00 AM

-----

This question was designed to highlight some of the oddities in VBScript's rules for what makes a legal identifier. 6) Which of the following are syntactically legal?  Why? (a) Explicit Error  
(b) For\[i=1\]=\[To\[1\]To\[1\]  
(c) For i =.For To Step Step Step  
(d) For Each Each In In  
  
(d) is illegal, the rest are legal. (a) is legal because neither Explicit nor Error are reserved words.  
(b) is legal because any text (of fewer than 255 characters, no newlines or \] allowed) surrounded by brackets is a legal identifier. In this case, \[i=1\], \[To\[1\] and \[1\] are legal identifiers. Yes, \[ is a legal character in a bracket identifier.  
(c) is legal because it is legal to use reserved words such as For after the dot operator, and because Step is not a reserved word.  
(d) is illegal because both Each and In are reserved. Notice that we can get away with making keywords unreserved if doing so does not introduce an ambiguity. However, we prefer to ensure not only that the program, statement or expression is unambiguous as a whole, but furthermore that the parser be able to disambiguate by looking forward no more than one token. If Each were a legal identifier, then (as one reader pointed out) For Each Foo In Bar and For Each = 1 To 10 require the parser to look ahead two from the For to determine what kind of loop this is. Step on the other hand can easily be disambiguated. Of course, so could In, which *is* reserved. My opinion is that when possible, one should err on the side of reserving, rather than allowing people to write patent nonsense like Step Step Step, but as we discussed last time, sometimes you can't reserve words that you'd like to.

