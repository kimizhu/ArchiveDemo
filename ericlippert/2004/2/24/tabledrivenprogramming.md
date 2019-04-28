# Table Driven Programming

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/24/2004 10:38:00 AM

-----

 

**Table driven programming **is a technique that can make some programs more readable and maintainable, but is often overlooked by script developers. 

Let me give you an example.  Here's a fragment of a VBScript program that I ran into yesterday.  It's perfectly correct, but it could be a lot shorter. 

Select Case wmiAce.AceType  
Case 0  
  OutFile.WriteLine "\<ACE type=""Access Allowed""\>"  
Case 1  
  OutFile.WriteLine "\<ACE type=""Access Denied""\>"  
Case 2   
  OutFile.WriteLine "\<ACE type=""Audit""\>"  
End Select  
If 1048576 And wmiAce.AccessMask Then   
  OutFile.WriteLine "  \<Perm\>Synchronize\</Perm\>"  
End If  
If 524288 And wmiAce.AccessMask Then  
  OutFile.WriteLine "  \<Perm\>Write Owner\</Perm\>"  
End If 

' \[... thirty more lines just like that ...\] 

If 2 And wmiACE.AccessMask Then  
  OutFile.WriteLine "  \<Perm\>File Write Data\</Perm\>"  
End If  
If 1 And wmiACE.AccessMask Then  
  OutFile.WriteLine "  \<Perm\>File Read Data\</Perm\>"  
End If  
OutFile.WriteLine "\</ACE\>" 

There's nothing broken about this program, and it's perfectly straightforward.  But it is so redundant\!  So repetitive\!  It says the same thing over and over.  It has the same pattern again and again.  It just goes on and on with only minor changes on each line.

Ahem. All that redundancy is just plain tiring to the eye, and the potential for typos and other mistakes goes up as the program gets longer.  If I were writing this program, I'd write it like this: 

AceType = Array("Access Allowed", "Access Denied", "Audit")  
OutFile.WriteLine "\<ACE type=""" & AceType(wmiAce.AceType & """\>"  
  
Access = Array("File Read Data", "File Write Data", \[...\] , "Synchronize")  
For Flag = LBound(Access) To UBound(Access)  
  If 2^Flag And wmiAce.AccessMask Then  
    OutFile.WriteLine "  \<Perm\>" & Access(Flag) & "\</Perm\>"  
  End If  
Next  
OutFile.WriteLine "\</ACE\>"\>

Which reduces a 51 line program to nine much more readable lines.  

Table driven programming is extremely powerful in JScript, because JScript supports sparse associative arrays and first class functions.  It's quite easy to define function tables in JScript.  For example: 

function handleRead() { ... }  
function handleWrite() { ... }    
// ... etc… 

var handlers = new Array();  
handlers\["Read"\] = handleRead;  
handlers\["Write"\] = handleWrite;  
// ... etc ... 

// later:  
var handler = handlers\[userInput\];  
if (handler \!= null)  
      handler(); 

Which is a lot more compact than the equivalent switch statement.  

Next time you build a really big conditional statement, ask yourself whether you could summarize all of the choices into a table.  It can make a program considerably more readable.

