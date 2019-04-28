# Error Handling in VBScript, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/19/2004 2:14:00 PM

-----

OK, enough about the Peloponnesian war -- a number of readers have asked me questions about error handling in VBScript recently, so I think I'll talk about it a bit for the next few days.

Today, I want to very carefully describe what the error handling semantics are in the language, because there is some confusion over how exactly it works. There are two statements that affect error handling in VBScript:

 

On Error Resume Next  
On Error Goto 0

The meaning of the first seems clear -- if you get an error, ignore it and resume execution on the next statement. But as we'll see, there are some subtleties. But before that, what the heck is up with the second statement?

The second statement turns off 'resume next' mode if it is on. Yes, the syntax is ridiculous -- something like On Error Raise would be a whole lot more clear. But for historical purposes, this is what we're stuck with. Visual Basic has an error handling mode which VBScript does not -- VB can branch to a labeled or numbered statement. (Remember line numbers? Those were the days\!) To tell VB that you no longer wish to branch to that statement, you give zero, an invalid line number. C'est super-beaucoup-de-fromage, n'est-ce pas? But we're stuck with it now.

The subtlety in the "resume next" mode is best illustrated with an example.

Const InvalidCall = 5  
Print "Global code start"  
Blah1  
Print "Global code end"  
Sub Blah1()  
      On Error Resume Next  
      Print "Blah1 Start"  
      Blah2  
      Print "Blah1 End"  
End Sub  
Sub Blah2()  
      Print "Blah2 Start"        
      Err.Raise InvalidCall  
      Print "Blah2 End"  
End Sub 

This prints out

 

Global code start  
Blah1 Start  
Blah2 Start  
Blah1 End  
Global code end

Hold on a minute -- when the error happened, Blah1 had already turned 'resume next' mode on. The next statement after the error raise is Print "Blah2 End" but that statement never got executed. What's going on?

What's going on is that **the error mode is on a per-procedure basis, not a global basis**. (If it were on a global basis, all kinds of bad things could happen -- think about how you'd have to design a program to have consistent error handling in a world where that setting is global, and you'll see why it's per-procedure.) In this case, Blah2 gets an error. Blah2 is not in 'resume next' mode, so it aborts itself, records that there was an error situation, and returns to its caller. The caller sees the error, but the caller is in 'resume next' mode, so it resumes.

In short, the **propagation model for errors in VBScript is basically the same as traditional structured exception handling** -- the exception is thrown up the stack until someone catches it, or the program terminates. However, the error information that can be thrown, and the semantics of the catcher are quite a bit weaker than, say, JScript's structured exception handling.

Also, remember that the 'next' in 'resume next' mode is the next **statement**. Consider these two programs, for example. Do they have the same semantics?

 

On Error Resume Next  
Temp = CInt(Foo.Bar(123))  
Blah Temp  
Print "Done"  
  
On Error Resume Next  
Blah CInt(Foo.Bar(123))  
Print "Done"

No\! If Foo.Bar raises an error, then the first one passes Empty to Blah. The second one never calls Blah at all if an error is raised, because it resumes to the next **statement**.

You can get into similar trouble with other constructs. For example, these **do** have the same semantics:

 

On Error Resume Next  
If Blah Then  
      Print "Hello"  
End If  
Print "goodbye"  
  
On Error Resume Next  
If Blah Then Print "Hello"  
Print "goodbye"  

If Blah raises an error then it resumes on the Print "Hello" **in either case**. You can also get into trouble with loops:

 

On Error Resume Next  
For index = 1 to Blah  
      Print TypeName(index)  
Next  
Print "Goodbye"

If Blah raises an error, this resumes **into** the loop, not **after** the loop. This prints out

 

Empty  
Goodbye

Be careful\! Next time I'll talk a bit about ways to avoid these gotchas, the semantics of the Err object, and general philosophies of error handling.

