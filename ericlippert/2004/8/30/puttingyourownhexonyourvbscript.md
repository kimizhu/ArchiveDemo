# Putting Your Own Hex On Your VBScript

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/30/2004 11:26:00 AM

-----

Here's a fragment of an email I received last week: bigValue = 4294967295  
Wscript.Echo Hex(bigValue)  
I would like this to display FFFFFFFF, but instead get the error "Overflow: bigValue What's up with that? The Hex function only works on numbers which fit into a four-byte signed integer. Since it's signed, you can have negative numbers in there. Negative numbers are represented in hex by their **complement**. Therefore, the number that produces FFFFFFFF is -1, not 4294967295. Therefore, the questioner is going to have to learn to live with disappointment -- you can't always get what you want. But of course, you should feel free to write your own Hex function that has whatever semantics you prefer.  For instance, here’s one that will hexify an arbitrary double, and uses the innovation of a negative sign to indicate negative numbers. It uses some cheesy math tricks to repeatedly get the modulus of the number and shift the number down by a factor of 16 until there's nothing left: Function MyHex(ByVal Number)  
  Dim Sign  
  Const HexChars = "0123456789ABCDEF"  
  Sign = Sgn(Number)  
  Number = Fix(Abs(CDbl(number)))  
  If Number = 0 Then  
    MyHex = "0"  
    Exit Function  
  End If  
  While Number \> 0  
    MyHex = Mid(HexChars, 1 + (Number - 16 \* Fix(Number / 16)), 1) & MyHex   
    Number = Fix(Number/16)  
  WEnd  
  If Sign = -1 Then MyHex = "-" & MyHex  
End Function  
  
All I ask is that you not call the function "Hex", and thereby overwrite the existing Hex function, unless you really, **really** want to and know what you're doing.  Pulling shens like that can come back to haunt you later; I once spent about a day debugging an ASP page on MSN.com before I realized that it had it's own personal buggy implementation of InStr for some crazy reason, and someone else who modified the page did not realize that.

