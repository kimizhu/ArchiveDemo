# VBScript Quiz Answers, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/9/2005 12:18:00 PM

-----

5\) Which of the following statements are guaranteed to never print False?  Assume that On Error Resume Next is in effect. (a) If x = x Then Print True Else Print False  
(b) If CStr(x) = CStr(x) Then Print True Else Print False  
(c) If True Or x Then Print True Else Print False  
(d) x = True : If x Then Print True Else Print False (c), as far as I know, always prints True.  The rest can be forced to print False. (a)   prints False if x is Null, because Null = Null produces Null, and Null coerces to False, so this takes the Else clause. (b) If x cannot be coerced to a string then On Error Resume Next resumes **into the consequence block of the conditional** -- bet you didn't know that\! -- so it prints "True". But what about this? Dim abc  
abc = 1  
Class Foo  
  Public Default Property Get Blah  
    Blah = abc  
    abc = abc + 1  
  End Property  
End Class  
Set x = new Foo Now x has different values when converted to a string on two occasions. I can't think of any way that (c) can go into the Else block. If x cannot be converted to a numeric then On Error Resume Next will resume into the consequence block. If it can, then Or'ing always produces True.  
  
(d) can produce False if x is read-only. Const x = False  
On Error Resume Next  
x = True : If X Then Print True Else Print False A number of readers pointed out other bizarre ways that (d) can produce False: Function X : X = False : End Function  
On Error Resume Next  
x = True : If X Then Print True Else Print False An answer I expected but did not get was the case where X is an object that has a default property, so the assignment is to the default property, not the object. That's not a valid case in VBScript, though it is in VB6. In VB6 the type system can detect when the "Set" is missing because in the VB6 type system, you can declare X as Object. In VBScript everything is Variant, so that trick doesn't work.

