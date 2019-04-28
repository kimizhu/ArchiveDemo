# VBScript Quiz Answers, Part Eight

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/23/2005 11:51:00 AM

-----

8\) Which of these programs is syntactically legal?

(a)  
Class Bar  
  Function Foo  
  End Function  
  Function Foo  
  End Function  
End Class (b)  
Class Bar  
  If Blah Then  
    Function Foo  
    End Function  
  Else  
    Function Foo  
    End Function  
  End If  
End Class (c)  
If Blah Then  
  Function Foo  
  End Function  
Else  
  Function Foo  
  End Function  
End If (d)  
Select Case Blah  
  Function Foo  
  End Function  
Case True  
  Function Foo  
  End Function  
End Select Only (c) is legal, the rest are illegal. None are legal in VB6. (a) is illegal because it is illegal to define the same procedure twice inside a class. (b) is illegal because *only* procedure and variable declarations are legal inside a class.  (d) is illegal because you cannot put a function between the Select and Case statements. There are three interesting facts about (d). First, it *is* legal to redefine aprocedure at the global level. Silly, but legal. Second, the syntax error raised by the parser is on the Case statement, bizarrely enough. Third, bizarrely enough it *is* legal to declare a procedure inside a Case, just not between a Select and a Case. (c) is legal. Due to a mistake in the VBScript 1.0 parser, we didn't check to see if functions were inside global-code If or Case statements, and we didn't check to see if they were redefined. This is totally an oversight -- it was not intentional. They're treated by the code generator as though the last one wins, and as though they were declared at global scope. There is no way to do "conditional compilation" here -- the last one wins no matter which branch of the conditional is executed. I wanted to fix this in VBScript 2.0, but we couldn't because we discovered that many ASP page authors accidentally put function declarations inside If blocks, and we didn't want to break them all. (We'll run into this problem again in \#10.) When I implemented classes in VBScript v5.0 of course there was no previous version of classes to be compatible with, so I tightened it down; a class may not redefine functions. And since a class scope does not allow conditional statements, the problem with functions inside conditionals goes away inside classes. Many of these quiz corner cases illustrate an important point: get the language design and implementation right the first time, because you're stuck with it forever\!

