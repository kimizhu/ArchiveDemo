# Why does JScript have rounding errors?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2003 1:41:00 PM

-----

 

Try this in JScript: 

window.alert(9.2 \* 100.0); 

You might expect to get 920, but in fact you get 919.9999999999999.  What the heck is going on here? 

Boy, have I ever heard this question a lot. 

Well, let me answer that question with another question.  Suppose you did a simple division, say 

window.alert(1.0 / 3.0); 

Would you expect an **infinitely large window **that said "0.33333333333..." with **an infinite number of threes**, or would you expect ten or so digits?  Clearly you'd expect it to not **fill your computer's entire memory with threes**.  But that means that I must ask **why are you willing to accept an error of 0.00000000000333333... in the case of dividing one by three but not willing to accept a smaller error of 0.000000000001 in the case of multiplying 9.2 by 100?**  

The simple fact **is that computer arithmetic frequently accumulates tiny errors.  **Any mathematics which results in numbers that cannot be represented by a small number of powers of two will result in errors.  

Let's look at this cases a little closer.  We're trying to multiply 9.2 by 100.0.  100.0 can be EXACTLY represented as a floating point number because it's an integer.  But 9.2 can't be -- you can't represent 46 / 5 exactly in base two any more than you can represent 1 / 3 exactly in base ten.  So when converting from the string "9.2" to the internal binary representation, a tiny error is accrued.  However, it's not all bad -- the 64 bit binary number which represents 9.2 internally is (a) the 64 bit float closest to 9.2, and (b) the algorithm which converts back and forth between strings and binary representation will **round-trip** -- that binary representation will be converted back to 9.2 if you try to convert it to a string. 

But now we go and throw a wrench in the works by multiplying by one hundred.  **That's going to lose the last few bits of precision because we just multiplied the *accrued error* by a factor of one hundred**.  The accrued error is now large enough that the string which most exactly represents the computed value is NOT "920" but rather "919.9999999999999".  

The ECMAScript specification mandates that JScript display **as much precision as possible** when displaying floating point numbers as strings.  

You may note that VBScript does not do this -- VBScript has heuristics which look for this situation and deliberately break the round-trip property in order to make this look better.  In JScript you are guaranteed that **when you convert back and forth between string and binary representations you lose no data**.  In VBScript you sometimes lose data; there are some legal floating point values in VBScript which are impossible to represent in strings accurately. In VBScript it is impossible to represent values like 919.9999999999999 precisely because they are automatically rounded\! 

 Ultimately, the reason that this is an issue is because we as human beings see numbers like "920" as SPECIAL.  If you multiply 3.63874692874  by 4.2984769284 and get a result which is one-billionth of one percent off, no one cares, but when you multiply 9.2 by 100.0 and get a result which is one-billionth off, everyone yells (at me\!)  **The computer doesn't know that 9.2 is more special than 3.63874692874   -- it uses the same lossy algorithms for both. **

 All languages which use floating point arithmetic have this feature -- C++, VBScript, JScript, whatever.  If you don't like it, either get in the habit of calling rounding functions, or don't use floating point arithmetic, use only integer arithmetic.  (Note that VBScript supports a "currency" type which is fixed-point number not subject to rounding errors.  It can only represent four places after the decimal point though.)

