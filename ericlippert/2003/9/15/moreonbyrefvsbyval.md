# More On ByRef vs ByVal

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2003 7:03:00 PM

-----

In my [previous entry](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx") I discussed VBScript's various syntaxes for passing values by reference.  However it occurs to me that there may be some confusion about what exactly "byref" and "byval" mean in JScript and VBScript.  This is frequently a source of confusion, as VBScript has byref behaviour not supported by JScript. 

The confusion arises because VBScript uses "by reference" to mean two similar, but different things.  VBScript supports **reference types** and **variable references**.  JScript supports the **former** but not the **latter**. 

The best way to illustrate the difference is with an example.  Consider this class: 

Class Foo   
      Public Bar  
End Class 

Now we can create an instance of this class: 

Dim Blah, Baz  
Set Blah = New Foo  
Set Baz = Blah  
Blah.Bar = 123 

Both Blah and Baz are references to the same object.  The fourth line changes both Blah.Bar and Baz.Bar because these are different names for the same thing. 

That's the "reference type" feature.  We say that VBScript treats objects as reference types. 

Now consider this little program: 

Sub Change(ByRef XYZ)   
    XYZ = 5  
End Sub  
Dim ABC  
ABC = 123  
Change ABC 

This passes a reference to variable ABC.  The local variable XYZ becomes an alias for ABC, so the assignment XYZ = 5 changes ABC as well. 

JScript has reference types -- all object types are reference types.  But **JScript does not support variable references**.  There is no way for one scope to change the value of a variable in another scope. 

UPDATE: articles on related topics can be found here:

<http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx>  
  
<http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx>  
  
<http://blogs.msdn.com/ericlippert/archive/2003/09/29/53117.aspx>

