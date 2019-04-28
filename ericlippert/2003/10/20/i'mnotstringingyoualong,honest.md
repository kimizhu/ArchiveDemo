# I'm not stringing you along, honest

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/20/2003 2:54:00 PM

-----

JScript and VBScript are often used to build large strings full of formatted text, particularly in ASP. Unfortunately, naïve string concatenations are a major source of performance problems. 

 

 

Before I go on, I want to note that it may seem like I am contradicting my earlier post, by advocating some "tips and tricks" to make string concatenation faster.  Do not blindly apply these techniques to your programs in the belief that they will magically make your programs faster\!  You always need to first determine what is fast enough, next determine what is not fast enough, and THEN try to fix it\!

 

 

JScript and VBScript use the aptly-named *naïve concatenation algorithm* when building strings. Consider this silly JScript example:

 

 

var str = "";

for (var count = 0 ; count \< 100 ; ++count)

  str = "1234567890" + str;

 

 

The result string has one thousand characters so you would expect that this would copy one thousand characters into str. Unfortunately, JScript has no way of knowing ahead of time how big that string is going to get and naïvely assumes that every concatenation is the *last* one.

 

 

On the first loop str is zero characters long. The concatenation produces a new ten-character string and copies the ten-character string into it. So far **ten** characters have been copied. On the **second** time through the loop we produce a new **twenty**-character string and copy two ten-character strings into it. So far 10 + 20 = 30 characters have been copied.

 

 

You see where this is going. On the **third** time **thirty** more characters are copied for a total of **sixty**, on the **fourth** **forty** more for a total of **one hundred**. Already the string is only forty characters long and we have copied more than twice that many characters. By the time we get up to the hundredth iteration **over fifty thousand characters have been copied to make that one thousand character string**. Also there have been ninety-nine temporary strings allocated and immediately thrown away. 

 

 

Moving strings around in memory is actually pretty darn fast, though 50000 characters is rather a lot.  Worse, allocating and releasing memory is not cheap.  OLE Automation has a string caching mechanism and the NT heap is pretty good about these sorts of allocations, but still, large numbers of allocations and frees are going to tax the performance of the memory manager.

 

 

If you're clever, there are a few ways to make it better.  (However, like I said before, always make sure you're spending time applying cleverness in the right place.)

 

 

One technique is to ensure that you are always concatenating small strings to small strings, large strings to large strings.  Pop quiz: what's the difference between these two programs?

 

 

for (var count = 0 ; count \< 10000 ; ++count)

  str += "1234567890" + "hello";

 

 

and

 

 

For count = 1 To 10000

  str = str & "1234567890" & "hello"

Next

 

 

?  I once had to debunk a widely distributed web article which claimed that VBScript was orders of magnitude slower than JScript because the comparison that the author used was to compare the above two programs.  Though they produce the same output, the JScript program is MUCH faster.  Why's that?  Because this is not an apples-to-apples comparison.  The VBScript program is equivalent to this JScript program:

 

 

for (var count = 0 ; count \< 10000 ; ++count)

  str = (str + "1234567890") + "hello";

 

 

whereas the JScript program is equivalent to this JScript program

 

 

for (var count = 0 ; count \< 10000 ; ++count)

  str = str + ("1234567890" + "hello");

 

 

See, the first program does two concatenations of a small string to a large string in one line, so the entire text of the large string gets moved twice every time through the loop.  The second program concatenates two small strings together first, so the small strings move twice but the large string only moves once per loop.  Hence, the first program runs about twice as slow.  The number of allocations remains unchanged, but the number of bytes copied is much lower in the second.

 

 

In hindsight, it might have been smart to add a multi-argument string concatenation opcode to our internal script interpreter, but the logic actually gets rather complicated both at parse time and run time.  I still wonder occasionally how much of a perf improvement we could have teased out by adding one.  Fortunately, as you'll see below, we came up with something better for the ASP case.

 

 

The other way to make this faster is to make the number of allocations smaller, which also has the side effect of not moving the bytes around so much. 

 

 

var str = "1234567890"; 10

str = str + str; 20

var str4 = str + str;  40

str = str4 + str4;  80

str = str + str;  160

var str32 = str + str;  320

str = str32 + str32; 640

str = str + str32 960

str = str + str4; 1000

 

 

This program produces the same result, but with 8 allocations instead of 100, and only moves 3230 characters instead of 50000+.   However, this is a rather contrived example -- in the real world strings are not usually composed like this\!

 

 

Now, those of you who have written programs in languages like C where strings are not first-class objects know how to solve this problem efficiently.  You build a buffer that is bigger than the string you want to put in, and fill it up.  That way the buffer is only allocated once and the only copies are the copies into the buffer.  If you don't know ahead of time how big the buffer is, then a double-when-full strategy is quite optimal -- pour stuff into the buffer until it's full, and when it fills up, create a new buffer twice as big.  Copy the old buffer into the new buffer and continue.  (Incidentally, this is another example of one of the "no worse than 200% of optimal" strategies that I was discussing earlier -- the amount of used memory is never more than twice the size of the memory needed, and the number of unnecessarily copied bytes is never more than twice the size of the final buffer.)

 

 

Another strategy that you C programmers probably have used for concatenating many small strings is to allocate each string a little big, and use the extra space to stash a pointer to the next string.  That way concatenating two strings together is as simple as sticking a pointer in a buffer.  When you're done all the concatenations, you can figure out the size of the big buffer you need, and do all the allocations and copies at once.  This is very efficient, wasting very little space (for the pointers) in common scenarios.

 

 

Can you do these sorts of things in script?  Actually, yes.  Since JScript has automatically expanding arrays you can implement a quick and dirty string builder by pushing strings onto an array, and when you're done, joining the array into one big string.  In VBScript it's not so easy because arrays are fixed-size, but you can still be pretty clever with fixed size arrays that are redimensioned with a "double when full" strategy.  But surely there is a better way than these cheap tricks.   

 

 

Well, in ASP there is.  You know, I used to see code like this all the time:

 

 

str = "\<blah\>"

str = str + blah

str = str + blahblah

str = str + whatever

 

 

' etc, etc, etc -- the string gets longer and longer, we have some loops, etc.

 

 

str = str + "\</blah\>"  
Response.Write str

 

 

Oh, the pain.  **The** Response** object is an efficient string buffer written in C++**.  Don't build up big strings, just dump 'em out into the HTML stream directly.  Let the ASP implementers worry about making it efficient.

 

 

"*Hold on just a minute. Mister Smartypants Lippert*," I hear you say, "*Didn't you just tell us last week that eliminating calls to COM objects is usually a better win than micro-optimizing small stuff like string allocations?*" 

 

 

Yes, I did.  But in this case, that advice doesn't apply because I know something you don't know.   

 

 

We realized that all the work that the ASP implementers did to ensure that the string buffer was efficient was being overshadowed by the inefficient late-bound call to Response.Write.  So we special-cased VBScript so that it detects when it is compiling code that contains a call to Response.Write and there is a named item in the global namespace called Response that implements IResponse::Write

