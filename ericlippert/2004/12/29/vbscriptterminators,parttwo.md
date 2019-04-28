# VBScript Terminators, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/29/2004 7:47:00 PM

-----

You guys came up with good answers to three of my four questions, which is about what I expected; question 2 was pretty hard. To sum up: QUESTION \#1: Why does the termination logic go terminate, terminate, terminate, clear, clear, clear, instead of terminate and clear, terminate and clear, terminate and clear? Because if the *second* object to be terminated has a terminator that accesses a property of the *first* object to be terminated and cleared, it will fail, which seems bad. We want to run all the terminators while the objects are still in good shape, and then blow them all away. QUESTION \#3: Why do we want to ensure that terminators don’t run twice? Imagine a terminator which writes a "logging complete" message to a log file; you don't want it to run twice. QUESTION \#4: Why do we run the garbage collector at the end of every statement, instead of only at the end of every procedure/global block? Because, though local variables are not going to go out of scope, temporary anonymous slots are. Consider x = MyFunc("a" + s, new Foo) That's going to allocate one temporary slot for the string and one for the object. The temporary string can be cleared whenever, but the temporary object should be released ASAP so that it's terminator runs ASAP. We therefore clean up all temporaries after every statement. That leaves QUESTION \#2: In what scenario can a bad implementation crash the process and/or terminate the object twice? When I first wrote the termination logic, the release code looked like this: ULONG VBSClassInstance::Release(){  
  --this-\>m\_cRef;  
  if (this-\>m\_cRef == 0)  
  {  
    this-\>RunTerminator();  
    delete this;  
    return 0;  
  }  
  return this-\>m\_cRef;  
} Looks like a perfectly straightforward implementation, right? But what if some bozo does this? Dim Global  
Class Foo  
  Private Sub Class\_Terminate()  
    Set Global = Me  
  End Sub  
End Class  
Sub Blah  
  Dim Local  
  Set Local = New Foo  
End Sub  
Blah  
Set Global = Nothing  
When Local goes out of scope, the terminator runs and sets the Global variable to the object which has just been terminated\! Therefore it must live. We must write the VBSClassInstance::RunTerminator method to ensure that the terminator doesn't run twice, but that's the least of our problems. Look at the implementation of Release above carefully. When the terminator runs, the ref count will go back up to one, but we still delete the object\! The script engine now has a global variable containing a pointer to deleted memory; this will crash the process, corrupt the heap, who knows what? OK, so what if we go ULONG VBSClassInstance::Release(){  
  --this-\>m\_cRef;  
  if (this-\>m\_cRef == 0)  
  {  
    this-\>RunTerminator();  
  }  
  if (this-\>m\_cRef == 0)  
  {  
    delete this;  
    return 0;  
  }  
  return this-\>m\_cRef;  
} Is that better? Well, sure, it's *better*, but it's still *wrong*. Forget globals; consider this:   Private Sub Class\_Terminate()  
    Dim TermLocal  
    Set TermLocal = Me  
  End Sub  
  
Now what happens?  The "final" release when Local goes out of scope sets the ref count to zero and calls the terminator. The terminator increases the ref count to one by assigning it. Then when TermLocal goes out of scope, it calls Release on the object. We've now got a re-entrant Release method\! The "inner" Release detects that the ref count has gone to zero and deletes the object. Then the "outer"  Release  reads from the now-invalid "this" pointer. Assuming that doesn't crash, it then probably corrupts the heap by releasing the object a second time. The correct logic looks something like this: ULONG VBSClassInstance::Release(){  
  --this-\>m\_cRef;  
  if (this-\>m\_cRef == 0)  
  {  
    ++this-\>m\_cRef;  
    // protects against re-entrant final release  
    this-\>RunTerminator();  
    --this-\>m\_cRef;  
    if (this-\>m\_cRef == 0)  
    {  
      delete this;  
      return 0;  
    }  
  }  
  return this-\>m\_cRef;  
} Writing correct shutdown logic is surprisingly tricky\!

