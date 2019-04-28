<div id="page">

# More On ByRef vs ByVal

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2003 7:03:00 PM

-----

<div id="content">

<div>

<span>In my [previous entry](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx") I discussed VBScript's various syntaxes for passing values by reference.  However it occurs to me that there may be some confusion about what exactly "byref" and "byval" mean in JScript and VBScript.  This is frequently a source of confusion, as VBScript has byref behaviour not supported by JScript. </span>

<span></span>

<span>The confusion arises because VBScript uses "by reference" to mean two similar, but different things.  VBScript supports **<span>reference types</span>** and **<span>variable references</span>**.  JScript supports the **<span>former</span>** but not the **<span>latter</span>**. </span>

<span></span>

<span>The best way to illustrate the difference is with an example.  Consider this class: </span>

<span></span>

<span>Class Foo   
</span><span>      Public Bar  
</span><span>End Class </span>

<span></span>

<span>Now we can create an instance of this class: </span>

<span></span>

<span>Dim Blah, Baz  
</span><span>Set Blah = New Foo  
</span><span>Set Baz = Blah  
</span><span>Blah.Bar = 123 </span>

<span></span>

<span>Both </span><span>Blah</span><span> and </span><span>Baz</span><span> are references to the same object.  The fourth line changes both </span><span>Blah.Bar</span><span> and </span><span>Baz.Bar</span><span> because these are different names for the same thing. </span>

<span></span>

<span>That's the "reference type" feature.  We say that VBScript treats objects as reference types. </span>

<span></span>

<span>Now consider this little program: </span>

<span></span>

<span>Sub Change(ByRef XYZ)   
</span><span>    XYZ = 5  
</span><span>End Sub  
</span><span>Dim ABC  
</span><span>ABC = 123  
</span><span>Change ABC </span>

<span></span>

<span>This passes a reference to variable </span><span>ABC</span><span>.  The local variable </span><span>XYZ</span><span> becomes an alias for </span><span>ABC</span><span>, so the assignment </span><span>XYZ = 5</span><span> changes </span><span>ABC</span><span> as well. </span>

<span></span>

<span>JScript has reference types -- all object types are reference types.  But **<span>JScript does not support variable references</span>**.  There is no way for one scope to change the value of a variable in another scope. </span>

<span></span><span>UPDATE: articles on related topics can be found here:</span>

<span></span><span><http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx>  
  
<http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx>  
  
<http://blogs.msdn.com/ericlippert/archive/2003/09/29/53117.aspx> </span>

<span></span>

</div>

</div>

</div>

