# Win Fabulous Prizes\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/10/2005 11:10:00 PM

-----

I want to do a long -- potentially very long -- series of blog articles about the fundamental theoretical nature of formal computer languages, and the practical problems in producing compilers for them. To kick this off in a lighthearted way on a Friday, here's a quiz that shows some of the weird "corner cases" that bedevil language designers. These are mostly completely contrived, unrealistic code snippets -- I would sincerely hope that no sane person actually programs like this. But you've got to think about nonsense like this when you're designing and implementing a new language. Some of these I've discussed in previous blog entries. And obviously all of these questions can be answered by just cut-n-pasting them into a Windows Script Host file and trying them out. But before you do that, **see where your intuition leads you**. Make guesses, and *then* see if you're correct -- you might be surprised\! (Heck, I've taken the whole compiler apart and put it back together again several times and *I* was surprised by some of these\!) Email me your answers, **explaining your reasoning** -- the contestant with the most correct (or entertaining) explanations gets a free *autographed* copy of my book "The VB.NET Code Security Handbook", just as soon as I get my next box of copies. (Former employees of the Windows Script Team are ineligible; Microsofties, please don't look at the comments in the source code. Void where prohibited. Decisions of judges, ie, me, are final. Only one prize shall be awarded.)  Next week I'll post the best answers with some discussion. 1) Which of the following are syntactically legal VBScript statements? Which are illegal? Why? (a) x = 10&987&654&321  
(b) x = 10&987&&654&321  
(c) x = 10&987&&654&&321  
(d) x = 10&987&&654&&&321 2) Which of the following are Integer, which are Long? Why? a = TypeName(32768)  
b = TypeName(32767 + 1)  
c = TypeName(-32767-1)  
d = TypeName(-32768) 3) Which of the following are syntactically legal/illegal statements? Why? (a) x.y  
(b) x. y  
(c) x . y  
(d) x .y  
(e) x \_  
      . y 4) Inside a class definition, which of the following are syntactically legal/illegal declarations? Why? (a) Private Default Sub Default()  
(b) Public Default Function Default  
(c) Public Default  
(d) Public Default Property Get Default(Property) 5) Which of the following statements are *guaranteed* to never print "False"?  Assume that "On Error Resume Next" is in effect. (a) If x = x Then Print True Else Print False  
(b) If CStr(x) = CStr(x) Then Print True Else Print False  
(c) If True Or x Then Print True Else Print False  
(d) x = True : If x Then Print True Else Print False 6) Which of the following are syntactically legal/illegal statements?  Why? (a) Explicit Error  
(b) For\[i=1\]=\[To\[1\]To\[1\]  
(c) For i =.For To Step Step Step  
(d) For Each Each In In 7) What does this silly program do? Why? On Error Resume Next  
For i = 1 To 1/0  
  Print "x" & i & "x"  
Next  
Print "Done" (a) prints xx, Done  
(b) prints x1x, Done  
(c) prints Done  
(d) Goes into a loop, producing x1x, x2x, etc. 8) Which of these programs is syntactically legal/illegal? Why? (a)  
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
End Select 9) Which of these statements is syntactically legal/illegal?  Why? (a) Const Z = -+-+-+-10  
(b) Const Y = 2 + 2  
(c) Const X = -.1e310  
(d) Dim W(+10) 10) Which of these statements is syntactically legal/illegal?  Why? (a) If Blah Then Foo Else Bar  
(b) If Blah Then If Foo Then Bar  
(c) If Blah Then Foo Bar Else Bar Foo  
(d) If Blah Then Foo Bar End If 11) X and Y are both Booleans.  Which of the following *always* assigns True?  Why? (a) Z = Not X Or Y = X Imp Y  
(b) Z = X Imp Y = Not X Or Y  
(c) Z = X Eqv Y = Not X XOr Y  
(d) Z = Not X XOr Y = X Eqv Y  12) Consider this program: s = "Eric read Æsop's Fables as a child"  
arr = Split(s, "e", -1, 1) arr is an array with how many elements? (a) 2  
(b) 3  
(c) 4  
(d) 5

