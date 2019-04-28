# VBScript Quiz Answers, Parts Nine and Ten

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/30/2005 10:25:00 AM

-----

9\) Which of these statements is syntactically legal?  Why? (a) Const Z = -+-+-+-10  
(b) Const Y = 2 + 2  
(c) Const X = .1e310  
(d) Dim W(+10) (a) is legal. The rest are illegal. (a) is legal because you can put as many unary positive or negative operators on top of a constant as you want, so long as it's a numeric constant. (b) is illegal; as I've discussed [before](http://blogs.msdn.com/ericlippert/archive/2003/10/21/53264.aspx), we do not support "constant folding" in VBScript.(c) is illegal because it's outside the range of a float.(d) is illegal because, bizarrely enough, dimension lists must be just straight integers, no operators whatsoever. Clearly VBScript is inconsistent insofar as how unary operators and constants work together. These inconsistencies are one those unfortunate corner cases that got missed when the parser was designed. 10) Which of these statements is syntactically legal?  Why? (a) If Blah Then Foo Else Bar  
(b) If Blah Then If Foo Then Bar  
(c) If Blah Then Foo Bar Else Bar Foo  
(d) If Blah Then Foo Bar End If (a) is illegal, the rest are legal. The VBScript single-line If-statement parser is all screwed up. Again, some of these were bugs in the first version that we couldn't fix without breaking compatibility, and some are just plain bugs that never got fixed for no particular reason. (b) is perfectly legal, nothing odd here. (a) is legal in VB6 but not in VBScript because of a mistake. The parser sees IF ID THEN ID ELSE... and tries to parse "ID ELSE" as a statement. The statement processor (correctly)thinks that the id is a procedure call and it tries to parse the argument list. The argument list processor should detect theElse and figure oh, I'm in anIf and the statement is done, but it doesn't figure that out, it just creates a syntax error.(c) is legal because in this case the argument list parser finds an argument list before the Else and doesn't freak out. A number of people pointed out that having both Foo Bar and Bar Foo in the same program is a little weird. Yes, it is\! But the question was what is *syntactically* legal, not what makes sense. And in fact, it is possible for this to work: Class ABC  
  Public Default Sub DEF(X)  
    Msgbox X.Name  
  End Sub  
  Public Name  
End Class  
Set Foo = New ABC  
Foo.Name = "Foo"  
Set Bar = New ABC  
Bar.Name = "Bar"  
Blah = True  
If Blah Then Foo Bar Else Bar Foo (d) is another mistake. The VB6 parser does not allow this, but VBScript does. As with \#8, we couldn't fix it when we found it because it would have broken existing ASP page and web pages.  The End If is ignored. Funny story: I actually fixed this one for a beta release of the engines and a certain influential news site was broken by the fix. Bizarrely enough they had a web page with this syntax in it. They said they would give the new version of IE a bad review if upgrading broke even a single one of their hundreds of thousands of pages\! Backwards compatibility is very, very important in the scripting world. Of course we rolled the change back, and there's a comment in the code now cautioning future maintenance programmers to never change it.

