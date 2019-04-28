# Runtime Typing in VBScript

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 1:43:00 PM

-----

A few short takes today: 

SimpleScript 

I wrote the name table logic over the weekend but it is not ready to post yet. For those of you following the conversation in [yesterday's comments](http://blogs.msdn.com/ericlippert/archive/2004/04/15/114094.aspx#114384)about whether I should use standard templates or roll my own, I ended up rolling my own non-template hash table.  I'll discuss that design decision in more detail when I actually post the code. 

Evangelism 

Those of you who read the [Joel On Software forum](http://discuss.fogcreek.com/joelonsoftware/ "http://discuss.fogcreek.com/joelonsoftware/") are undoubtedly familiar with the witty and urbane postings of Philo.  You might not know that in his day job, he's an evangelist for, among other things, [VSTO](/vsto "http://blogs.msdn.com/vsto").  I'm looking forward to seeing what he has to say about the Office dev community on his [blog](/philoj "http://blogs.msdn.com/philoj"). 

Runtime Typing in VBScript 

Speaking of Joel, he was kind enough to put a link to my blog on his [front page](http://www.joelonsoftware.com/items/2004/04/16.html "http://www.joelonsoftware.com/items/2004/04/16.html").  Thanks Joel\!  

Joel and I got into a discussion recently about VBScript's ambiguous use of [parens](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx).  Joel asked: 

In the VBScript parser, if you have a = b(3) you don't know if b() is a function call or an array lookup until you run the code because b is a variant… how do you compile such a thing? 

Indeed, this situation is ambiguous and we do no type inference at compile time.  We can tell declared names from undeclared names and optimize accordingly, but we don't tell whether something is an array, a procedure or a method call on an object.  (Remember, if b is a collection, this might be sugar for b.item(3) ) 

VBScript compiles to a simple proprietary bytecode format which is then run through an interpreter.  Consider this program: 

Dim Aaa(10)  
Dim Bbb  
Bbb = Array(123)  
Function Ddd(z)  
End Function  
x = Aaa(0)  
x = Bbb(0)  
x = Ccc(0)  
x = Ddd(0)

The first two assignments to x get compiled into something like this bytecode: 

BOS              Beginning of statement  
IntConst  0      Push argument on stack  
CallLocal 1 1    Call local variable \#1 (Aaa), with one argument -- pops arguments, pushes result.  
StoreNamed 'x'   Pop the top of stack into variable 'x' 

Since we cannot find a dimensioned variable corresponding to the last two, both generate something like this instead of the local call: 

CallNamed 'Ccc' 1 

Which is [slightly slower](/ericlippert/archive/2003/10/17/53237.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx") but logically the same. Figuring out what to do happens entirely at runtime.  First, we resolve the local variable reference or name into some kind of variant.  We then check to see which of these situations we're in: 

  - it's a Function, Property Get or Sub, in which case we call the procedure
  - a method on a global named item -- think alert in IE, which is actually window.alert  -- which we then invoke
  - an ordinary dispatch object, in which case we invoke its default method
  - an array of appropriate rank, in which case we dereference the array
  - anything else: type mismatch error

We could, I suppose, deduce that a declared variable is typed as an array, but we don't.  Doing so would save hardly any time, as the check to see if a given variant is an object or an array is extremely simple, and it would complicate the code generator.

And I know what you're going to ask -- no, there is no publically available utility program that dumps the bytecode, sorry\!

