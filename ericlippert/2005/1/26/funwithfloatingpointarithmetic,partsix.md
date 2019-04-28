# Fun With Floating Point Arithmetic, Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/26/2005 1:02:00 PM

-----

One more thing -- I said earlier that the VBScript float-to-string algorithm was a little bit different than the JScript algorithm. We can demonstrate quite easily by comparing the outputs of two nigh-identical programs: ' VBScript  
print 9.2 \* 100.0 \< 920.0  
print 919.9999999999999 \< 920.0  
print 920.0000000000001 \> 920.0 ' JScript  
print(9.2\*100.00 \< 920.0);  
print(919.9999999999999 \< 920.0);  
print (920.0000000000001 \> 920.0); As you'd expect, the last two comparisons of each result in true. But why does the first also result in true? Because of the very issues we've been talking about in the last five parts -- 9.2 cannot be exactly represented as a float. There is some representation error. When the float is multiplied by 100, the representation error also gets 100 times larger, and that's big enough to make it slightly smaller than 920. If that's the case then why do these programs produce different output? print 9.2 \* 100.0  
print 919.9999999999999  
print 920.0000000000001 print(9.2\*100.00);  
print(919.9999999999999);  
print(920.0000000000001); The VBScript program produces 920, 920, 920. The JScript program produces 919.9999999999999, 919.9999999999999, 920.0000000000001. What is up with that? The JScript algorithm for converting floats to strings is designed to have as much precision as possible. Since 919.9999999999999 and 920.0 have different binary representations as floats, they have different string representation. The VBScript algorithm on the other hand assumes that if you have 919.9999999999999 or 920.0000000000001, that probably what has happened is you've run into a floating point error accrual issue, and it rounds it back to the correct value for you when it displays the string. This heuristic means that VBScript (paradoxically) loses a small amount of precision and yet displays more accurate results for typical cases. The down side is that VBScript is unable to display full precision when you really DO want to represent 919.9999999999999. Such cases are quite rare though, and the error created in such cases is tiny.

