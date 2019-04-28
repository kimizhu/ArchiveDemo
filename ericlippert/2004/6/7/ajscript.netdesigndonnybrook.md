# A JScript .NET Design Donnybrook

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/7/2004 2:15:00 PM

-----

When you're designing a new programming language, the "main line" cases are easy.  It's the "corner" cases that bedevil language designers -- the little fiddly bits where two language features that make perfect sense on their own interact in some weird way.  Unfortunately, I suspect that any language sufficiently feature-rich to be interesting will have weird corner cases.  Getting them right is one of the more interesting and difficult aspects of language design. 

Want to give it a try yourself?  Now's your chance.  Here is a question that sparked a protracted and hilarious debate amongst the JScript .NET design team back in 2000.  I'd be interested to see what your opinions are.  So as to not spoil the fun, **figure out what you think this code should do *before* you run it or decompile it into IL. **

class foo {  
  function get abc() {  
    return 5;  
  }  
}  
class foo2 extends foo {  
  function bar(ob) {  
    print(ob.abc);  
  }  
}  
class foo3 extends foo2 {   
   hide function get abc() {  
    return 50;  
  }  
}  
var f = new foo3(); 

Clearly print(f.abc); must print out 50.  But should f.bar(f); print out 50 or 5?   Justify your answer. 

HINT: While you're thinking it over, keep in mind that nowhere in this code are there any type annotations.  The question was originally raised in the context of speculative compiler optimizations which we might perform in the absence of type annotations.  Are there codegen optimizations which we could perform that make your desired behaviour more efficient in common cases?

