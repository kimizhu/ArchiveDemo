# "Finally" Does Not Mean "Immediately"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/1/2004 9:51:00 AM

-----

All that talk a while back about [error handling in VBScript](http://blogs.msdn.com/ericlippert/archive/2004/08/19/217244.aspx)got me thinking about some error handling security issues that involve VB.NET specifically, though they apply to managed code written in any language. Consider this scenario: you provide a **fully trusted library** of useful functions which can be used by **partially trusted callers**.  The partially trusted callers might be [**hostile**](http://blogs.msdn.com/ericlippert/archive/2003/09/25/53097.aspx)-- that's why they're partially trusted, after all.  You have a particular method which needs administration rights.  When it's called, it prompts the user for the administrator password, impersonates the administrator, does work, and then reverts the impersonation. Public Sub DoTheThing()  
  ' Omitted -- prompt user to obtain password  
  NewIdentity = GetWindowsIdentity(AdminName, Domain, Password)  
  NewContext = NewIdentity.Impersonate()  
  DoTheAdminThing()  
  NewContext.Undo()  
End Sub There's a potentially huge security hole here.  What if DoTheAdminThing throws an exception?  There's no Catch or Finally block around the Undo, so **control can return to the partially trusted caller with the impersonation still intact\!**  That seems bad\!  Let's stick a Finally on there:   NewIdentity = GetWindowsIdentity(UserName, Domain, Password)  
  NewContext = NewIdentity.Impersonate()  
  Try  
    DoTheAdminThing()   
  Finally  
    NewContext.Undo()  
  End Try Clearly this an improvement from a code-quality perspective, but does it solve the security problem? No\!  A Finally block guarantees that the code will run *eventually*, but it **does not guarantee** **that** **no code will run between the point of the exception and the Finally block\!**  VB .NET provides a syntax specifically for running code before the Finally block: the **exception filter**.  Suppose our hostile code looked like this: Public Sub IAmSoEvil()  
  Try  
    NiceObject.DoTheThing()  
  Catch Ex As System.Exception When ExploitFlaw()  
    DoSomethingElse()  
  End Try  
End Sub  
Public Function ExploitFlaw() As Boolean  
  DoSomethingEvil()   
  ExploitFlaw = True  
End Function Let's trace through what happens.

  - First, the hostile partially trusted 

IAmSoEvil calls the fully trusted DoTheThing method.

DoTheThing

impersonates the administrator and calls DoTheAdminThing.

DoTheAdminThing

throws an exception (possibly because the partially trusted code has previously eaten up lots of memory or otherwise caused error conditions to become more likely.)

The .NET Runtime finds the Catch block in

IAmSoEvil and runs ExploitFlaw to see if this block can handle the exception.

ExploitFlaw

calls DoSomethingEvil, which attempts to take advantage of the fact that it is now impersonating an administrator.

DoSomethingEvil

finishes its attempt at an exploit and returns.

ExploitFlaw

returns True. The .NET Runtime stops its search for exception handlers.

The .NET Runtime runs the code in the

Finally block of DoTheThing. The admin impersonation is reverted.

The .NET Runtime runs

DoSomethingElse in the Catch block of IAmSoEvil. Of course, this doesn't apply to just thread impersonation**.**  **Any time that important global state can be made inconsistent requires very careful exception handling to ensure that no hostile code runs while the state is vulnerable.**  We can do that by catching the error before the hostile code can: Dim Reverted As Boolean  
Reverted = False  
NewContext = NewIdentity.Impersonate()  
Try   
  DoTheAdminThing()  
Catch Ex As Exception  
  NewContext.Undo()  
  Reverted = True  
  Throw Ex  
Finally   
  If Not Reverted Then   
    NewContext.Undo()  
  End If  
End Try Gross, but there's not much else you can do about it.  Fortunately, these kinds of situations are pretty rare. Pop quiz: Is this another example of the flaw pattern? FileIOPermission.Assert()  
Try   
  DoSomething()  
Finally   
  CodeAccessPermission.RevertAssert()  
End Try Does this do the same thing as before -- potentially pass control to a partially-trusted exception filter before the assertion is reverted?  If so, how do you defend against it?  If not, why not, and are there any other potential attacks here?  Tune in next time to find out\! I'll be AFK for the Labour Day weekend, so we'll have the thrilling conclusion next week.  Have a nice weekend, everyone.

