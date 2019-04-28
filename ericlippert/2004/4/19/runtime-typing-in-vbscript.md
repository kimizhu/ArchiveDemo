<div id="page">

# Runtime Typing in VBScript

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 1:43:00 PM

-----

<div id="content">

<span>A few short takes today: </span>

<span></span>

<span>SimpleScript </span>

<span></span>

<span>I wrote the name table logic over the weekend but it is not ready to post yet. For those of you following the conversation in [yesterday's comments](http://blogs.msdn.com/ericlippert/archive/2004/04/15/114094.aspx#114384) about whether I should use standard templates or roll my own, I ended up rolling my own non-template hash table.  I'll discuss that design decision in more detail when I actually post the code. </span>

<span></span>

<span>Evangelism </span>

<span></span>

<span>Those of you who read the [Joel On Software forum](http://discuss.fogcreek.com/joelonsoftware/ "http://discuss.fogcreek.com/joelonsoftware/") are undoubtedly familiar with the witty and urbane postings of Philo.  You might not know that in his day job, he's an evangelist for, among other things, [VSTO](/vsto "http://blogs.msdn.com/vsto").  I'm looking forward to seeing what he has to say about the Office dev community on his [blog](/philoj "http://blogs.msdn.com/philoj"). </span>

<span></span>

<span>Runtime Typing in VBScript </span>

<span></span>

<span>Speaking of Joel, he was kind enough to put a link to my blog on his [front page](http://www.joelonsoftware.com/items/2004/04/16.html "http://www.joelonsoftware.com/items/2004/04/16.html").  Thanks Joel\!  </span>

<span></span>

<span>Joel and I got into a discussion recently about VBScript's ambiguous use of [parens](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx).  Joel asked: </span>

<span></span>

<span>In the VBScript parser, if you have</span><span> </span><span>a = b(3)</span><span> </span><span>you don't know if </span><span>b()</span><span> is a function call or an array lookup until you run the code because </span><span>b</span><span> is a variant… how do you compile such a thing? </span>

<span></span>

<span>Indeed, this situation is ambiguous and we do no type inference at compile time.  We can tell declared names from undeclared names and optimize accordingly, but we don't tell whether something is an array, a procedure or a method call on an object.  (Remember, if </span><span>b</span><span> is a collection, this might be sugar for </span><span>b.item(3)</span><span> ) </span>

<span>VBScript compiles to a simple proprietary bytecode format which is then run through an interpreter.  </span><span>Consider this program: </span>

<span></span>

<span>Dim Aaa(10)  
</span><span>Dim Bbb  
</span><span>Bbb = Array(123)  
</span><span>Function Ddd(z)  
</span><span>End Function  
</span><span>x = Aaa(0)  
</span><span>x = Bbb(0)  
</span><span>x = Ccc(0)  
</span><span>x = Ddd(0)</span>

<span></span>

<span>The first two assignments to x get compiled into something like this bytecode: </span>

<span></span>

<span>BOS              Beginning of statement  
</span><span>IntConst  0      Push argument on stack  
</span><span>CallLocal 1 1    Call local variable \#1 (Aaa), with one argument -- pops arguments, pushes result.  
</span><span>StoreNamed 'x'   Pop the top of stack into variable 'x' </span>

<span></span>

<span>Since we cannot find a dimensioned variable corresponding to the last two, both generate something like this instead of the local call: </span>

<span></span>

<span>CallNamed 'Ccc' 1 </span>

<span></span>

<span>Which is [slightly slower](/ericlippert/archive/2003/10/17/53237.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx") but logically the same. Figuring out what to do happens entirely at runtime.  First, we resolve the local variable reference or name into some kind of variant.  We then check to see which of these situations we're in: </span>

<span></span>

  - <span>it's a </span><span>Function</span><span>, </span><span>Property</span><span> </span><span>Get</span><span> or </span><span>Sub</span><span>, in which case we call the procedure</span>
  - <span></span><span>a method on a global named item -- think </span><span>alert</span><span> in IE, which is actually </span><span>window.alert  -- which we then invoke</span>
  - <span></span><span>an ordinary dispatch object, in which case we invoke its default method</span>
  - <span></span><span>an array of appropriate rank, in which case we dereference the array</span>
  - <span></span><span>anything else: type mismatch error</span>

<span>We could, I suppose, deduce that a declared variable is typed as an array, but we don't.  Doing so would save hardly any time, as the check to see if a given variant is an object or an array is extremely simple, and it would complicate the code generator.</span>

<span></span>

<span>And I know what you're going to ask -- no, there is no publically available utility program that dumps the bytecode, sorry\!</span>

<span></span> 

</div>

</div>

