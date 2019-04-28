<div id="page">

# Table Driven Programming

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/24/2004 10:38:00 AM

-----

<div id="content">

<div>

<span> </span>

<span></span>

**<span>Table driven programming </span>**<span>is a technique that can make some programs more readable and maintainable, but is often overlooked by script developers. </span>

<span></span>

<span>Let me give you an example.  Here's a fragment of a VBScript program that I ran into yesterday.  It's perfectly correct, but it could be a lot shorter. </span>

<span></span>

<span>Select Case wmiAce.AceType  
</span><span>Case 0  
</span><span>  OutFile.WriteLine "\<ACE type=""Access Allowed""\>"  
</span><span>Case 1  
</span><span>  OutFile.WriteLine "\<ACE type=""Access Denied""\>"  
</span><span>Case 2   
</span><span>  OutFile.WriteLine "\<ACE type=""Audit""\>"  
</span><span>End Select  
</span><span>If 1048576 And wmiAce.AccessMask Then   
</span><span>  OutFile.WriteLine "  \<Perm\>Synchronize\</Perm\>"  
</span><span>End If  
</span><span>If 524288 And wmiAce.AccessMask Then  
</span><span>  OutFile.WriteLine "  \<Perm\>Write Owner\</Perm\>"  
</span><span>End If </span>

<span></span>

<span>' \[... thirty more lines just like that ...\] </span>

<span></span>

<span>If 2 And wmiACE.AccessMask Then  
  </span><span>OutFile.WriteLine "  \<Perm\>File Write Data\</Perm\>"  
</span><span>End If  
</span><span>If 1 And wmiACE.AccessMask Then  
</span><span>  OutFile.WriteLine "  \<Perm\>File Read Data\</Perm\>"  
</span><span>End If  
</span><span>OutFile.WriteLine "\</ACE\>" </span>

<span></span>

<span>There's nothing broken about this program, and it's perfectly straightforward.  But it is so redundant\!  So repetitive\!  It says the same thing over and over.  It has the same pattern again and again.  It just goes on and on with only minor changes on each line.</span>

<span>Ahem. All that redundancy is just plain tiring to the eye, and the potential for typos and other mistakes goes up as the program gets longer.  If I were writing this program, I'd write it like this: </span>

<span></span>

<span>AceType = Array("Access Allowed", "Access Denied", "Audit")  
</span><span>OutFile.WriteLine "\<ACE type=""" & AceType(wmiAce.AceType & """\>"  
</span><span>  
</span><span>Access = Array("File Read Data", "File Write Data", \[...\] , "Synchronize")  
</span><span>For Flag = LBound(Access) To UBound(Access)  
</span><span>  If 2^Flag And wmiAce.AccessMask Then  
</span><span>    OutFile.WriteLine "  \<Perm\>" & Access(Flag) & "\</Perm\>"  
</span><span>  End If  
</span><span>Next  
</span><span>OutFile.WriteLine "\</ACE\>"\></span>

<span></span>

<span>Which reduces a 51 line program to nine much more readable lines.  </span>

<span></span>

<span>Table driven programming is extremely powerful in JScript, because JScript supports sparse associative arrays and first class functions.  It's quite easy to define function tables in JScript.  For example: </span>

<span></span>

<span>function handleRead() { ... }  
</span><span>function handleWrite() { ... }    
</span><span>// ... etc… </span>

<span></span>

<span>var handlers = new Array();  
</span><span>handlers\["Read"\] = handleRead;  
</span><span>handlers\["Write"\] = handleWrite;  
</span><span>// ... etc ... </span>

<span></span>

<span>// later:  
</span><span>var handler = handlers\[userInput\];  
</span><span>if (handler \!= null)  
</span><span>      handler(); </span>

<span></span>

<span>Which is a lot more compact than the equivalent </span><span>switch</span><span> statement.  </span>

<span></span>

<span>Next time you build a really big conditional statement, ask yourself whether you could summarize all of the choices into a table.  It can make a program considerably more readable.</span>

</div>

</div>

</div>

