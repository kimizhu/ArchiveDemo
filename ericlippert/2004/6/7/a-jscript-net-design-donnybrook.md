<div id="page">

# A JScript .NET Design Donnybrook

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/7/2004 2:15:00 PM

-----

<div id="content">

<div>

<span></span>

<span>When you're designing a new programming language, the "main line" cases are easy.  It's the "corner" cases that bedevil language designers -- the little fiddly bits where two language features that make perfect sense on their own interact in some weird way.  Unfortunately, I suspect that any language sufficiently feature-rich to be interesting will have weird corner cases.  Getting them right is one of the more interesting and difficult aspects of language design. </span>

<span></span>

<span>Want to give it a try yourself?  Now's your chance.  Here is a question that sparked a protracted and hilarious debate amongst the JScript .NET design team back in 2000.  I'd be interested to see what your opinions are.  So as to not spoil the fun, **<span>figure out what you think this code should do *<span>before</span>* you run it or decompile it into IL. </span>**</span>

<span></span>

<span></span>

<span>class foo </span><span>{  
</span><span>  function get abc()</span><span> {  
</span><span>    return 5;  
</span><span>  }  
</span><span>}  
</span><span>class foo2 extends foo </span><span>{  
</span><span>  function bar(ob)</span><span> {  
</span><span>    print(ob.abc);  
</span><span>  }  
</span><span>}  
</span><span>class foo3 extends foo2 </span><span>{</span><span>   
   hide function get abc()</span><span> {  
</span><span>    return 50;  
</span><span>  }  
</span><span>}  
</span><span>var f = new foo3(); </span>

<span></span>

<span>Clearly </span><span>print(f.abc); </span><span>must print out </span><span>50</span><span>.  But should </span><span>f.bar(f); </span><span>print out </span><span>50</span><span> or </span><span>5</span><span>?  </span> <span>Justify your answer. </span>

<span></span>

<span>HINT: While you're thinking it over, keep in mind that nowhere in this code are there any type annotations.  The question was originally raised in the context of speculative compiler optimizations which we might perform in the absence of type annotations.  Are there codegen optimizations which we could perform that make your desired behaviour more efficient in common cases?</span>

</div>

</div>

</div>

