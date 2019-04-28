# You Can't Convert Data Structures To Strings In VBScript Without Breaking A Few Eggs

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/25/2004 2:06:00 PM

-----

Here's a question I get every now and then:  

> I've written a VBScript program which calls a method on an object that returns an array of bytes containing a GUID.  VBScript only supports arrays of variants.  How can I turn this into a human-readable string?

 Good question.  It is doable without writing an object in C++, but it's a little tricky.  The first thing to know is that **even though VBScript does not support arrays of anything other than variant, the underlying OLE Automation library supports turning byte arrays into strings**.  Therefore you can use CStr to turn the thing into a string, right? 

 Function GuidToString(ByteArray)  
  GuidToString = CStr(ByteArray)  
End Function 

Print GuidToString(MyObject.GetTheGuid) 

 Which prints out ~å ??ATErU%'èÅp± 

 Oops.  We've taken those bytes and interpreted them as Unicode characters in a UTF-16 encoding.  That's not right.  We want to convert the bytes to text, preferably in hex format.  Fortunately we have it in a string now, so we can extract the bytes with the byte-manipulating versions of the string library functions. Let's try that again. 

 Function GuidToString(ByteArray)  
  Dim Binary, S  
  Binary = CStr(ByteArray)  
  S = "{"  
  S = S & Hex(AscB(MidB(Binary, 1, 1)))  
  S = S & Hex(AscB(MidB(Binary, 2, 1)))  
  S = S & Hex(AscB(MidB(Binary, 3, 1)))  
  S = S & Hex(AscB(MidB(Binary, 4, 1)))  
  S = S & "-"    
  S = S & Hex(AscB(MidB(Binary, 5, 1)))  
  S = S & Hex(AscB(MidB(Binary, 6, 1)))  
  S = S & "-"    
  S = S & Hex(AscB(MidB(Binary, 7, 1)))  
  S = S & Hex(AscB(MidB(Binary, 8, 1)))  
  S = S & "-"    
  S = S & Hex(AscB(MidB(Binary, 9, 1)))  
  S = S & Hex(AscB(MidB(Binary, 10, 1)))  
  S = S & "-"    
  S = S & Hex(AscB(MidB(Binary, 11, 1)))  
  S = S & Hex(AscB(MidB(Binary, 12, 1)))  
  S = S & Hex(AscB(MidB(Binary, 13, 1)))  
  S = S & Hex(AscB(MidB(Binary, 14, 1)))  
  S = S & Hex(AscB(MidB(Binary, 15, 1)))  
  S = S & Hex(AscB(MidB(Binary, 16, 1)))  
  S = S & "}"  
  GuidToString = S  
End Function 

 Which prints out {7E0E50-200-BA25-4026-410540450} 

 Uh, shouldn't the character counts of each section be 8-4-4-4-12, instead of 6-3-4-4-9 ?   Oops.  We need the single digit bytes like 0 to go to "00", not "0".  That's easy enough to fix up: 

 Function HexByte(b)  
      HexByte = Right("0" & Hex(b), 2)  
End Function 

 Function GuidToString(ByteArray)  
  Dim Binary, S  
  Binary = CStr(ByteArray)  
  S = "{"  
  S = S & HexByte(AscB(MidB(Binary, 1, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 2, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 3, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 4, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 5, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 6, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 7, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 8, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 9, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 10, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 11, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 12, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 13, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 14, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 15, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 16, 1)))  
  S = S & "}"  
  GuidToString = S  
End Function 

 Which prints out {7E00E500-2000-BA25-4026-410054004500} 

 Which is also wrong.  What's wrong this time?  

 **The logical format of a GUID in memory is not in the same order as the bytes are in the string.   **A GUID stored in binary format in memory is a sixteen byte structure in the following format: 

 DWORD-WORD-WORD-BYTE BYTE-BYTE BYTE BYTE BYTE BYTE BYTE 

 So what?  Why does that matter? 

 It matters because a WORD consists of two bytes, but they are stored in memory in order from the least to the most significant on my Intel machine.  Same with the four-byte DWORD.  Intel boxes are "little endian" machines.  Motorolas are "big endian" -- on Macs, the big byte comes first in memory.  Which is the better scheme is one of the great holy wars of information technology.  Apparently some poor deluded people still fail to realize that little-endian architecture is much more sensible than big-endian, or that vi is a much better editor than emacs. J 

 (ASIDE: These whimsical terms were borrowed from Gulliver's Travels, in which Swift satirizes the political parties of his day.  In Lilliput, the Protestant rulers of England are represented by the Little Endians, the oppressed Catholics as the Big Endians.  They disagree on which is the correct way to break an egg.  See the last half of part one, [chapter four](http://www.jaffebros.com/lee/gulliver/bk1/chap1-4.html)for details.) 

 We need to decode that thing into the correct order: 

 Function GuidToString(ByteArray)  
  Dim Binary, S  
  Binary = CStr(ByteArray)  
  S = "{"  
  S = S & HexByte(AscB(MidB(Binary, 4, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 3, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 2, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 1, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 6, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 5, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 8, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 7, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 9, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 10, 1)))  
  S = S & "-"    
  S = S & HexByte(AscB(MidB(Binary, 11, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 12, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 13, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 14, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 15, 1)))  
  S = S & HexByte(AscB(MidB(Binary, 16, 1)))  
  S = S & "}"  
  GuidToString = S  
End Function 

 Which prints out {00E5007E-0020-25BA-4026-410054004500}, the correct string. 

[The whole point of script programming languages is to abstract away](http://weblogs.asp.net/ericlippert/archive/2004/03/01/82168.aspx)from the underlying details of how the machine works.  Occasionally though these abstractions prove to be [leaky](http://www.joelonsoftware.com/articles/LeakyAbstractions.html "http://www.joelonsoftware.com/articles/LeakyAbstractions.html"). This is one of those times when in order to make sense of something, you need to understand some pretty low-level trivia about how computers work.

