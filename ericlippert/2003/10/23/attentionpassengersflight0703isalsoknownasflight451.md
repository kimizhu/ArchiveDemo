# Attention passengers: Flight 0703 is also known as Flight 451

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/23/2003 12:13:00 PM

-----

I hate octal.  Octal causes bugs.  I hate bugs, particularly stupid "gotcha" bugs.

 

 

Foolish C programmers do things like

 

 

int arr\_flight = 0703;

 

 

not realizing that this does not assign the number 703, but rather 7 x 64 + 3 = 451.

 

 

Even worse, foolish JScript programmers do things like

 

 

var arr\_flight = 0708;

var dep\_flight = 0707;

 

 

not realizing that **the former is a decimal literal** but **the latter is an octal literal**.  

 

 

Yes, in JScript it really is the case that if a literal begins with 0, consists of only digits and contains an 8 or a 9 then it is decimal but if it contains no 8 or 9 then it is octal\!  The first version of the JScript lexer did not implement those rules, but eventually we changed it to be compatible with Netscape's implementation.  

 

 

This is in keeping with the design principle that I mentioned earlier, namely **"Got a problem? Muddle on through\!"**  However, since this problem can be caught **at compile time**, I think that the decision to make illegal octal literals into decimals was a poor one.  

 

 

It's just a mess. Octal literals and escape sequences have been removed from the ECMAScript specification, though of course they live on in actual implementations for backwards compatibility.

 

 

This is why I added code to JScript .NET so that any use of an integer decimal or octal literal that begins with zero yields a compiler warning, with one exception.  Obviously x = 0; does not produce a warning\!

