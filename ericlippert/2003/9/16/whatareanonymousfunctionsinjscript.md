# What Are "Anonymous Functions" In JScript?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 7:57:00 PM

-----

One of our excellent customer support staff in the United Kingdom asked me this morning what "anonymous functions" are in JScript.

 

 

It's a little complicated. Two things come to mind: realio-trulio anonymous functions, and what we used to call "scriptlets".

 

 

First up are actual anonymous functions -- functions which do not have names are, unsurprisingly enough, called "anonymous functions".

 

 

"**What the heck do you mean, functions without names**?" I hear you ask.  "**Surely all functions have names\!"** -- well, no, actually, some don't.  This is perfectly legal in JScript:

 

 

print ( function(x){return x \* 2;} (3) );

 

 

That prints out "6".  What's the name of that function?  It has no name.  Of course, we could give it one if we chose.  This is exactly the same as declaring a named function:

 

 

function double(x){return x \* 2;} 

print ( double(3) );

 

 

But functions don't need names any more than strings or numbers do.  Functions are just functions whether they’re named or not.

 

 

Remember, JScript is a functional language.  Functions are objects and can be treated like any other object.  You don't have to give an object a name to use it, or you can give them multiple names.  Functions are no different.  For example, you can assign them to variables.

 

 

var myfunc = function(x){return x \* 2;} 

var myfunc2 = myfunc; 

print ( myfunc(3) );

 

 

Those of you who are familiar with more traditional functional languages, such as Lisp or Scheme, will recognize that functions in JScript are fundamentally the Lambda Calculus in fancy dress.  (The august Waldemar Horwat -- who was at one time the lead Javascript developer at AOL-Time-Warner-Netscape -- once told me that he considered Javascript to be just another syntax for Common Lisp.  I'm pretty sure he was being serious; Waldemar's a hard core language guy and a heck of a square dancer to boot.)   

 

 

Anyway, I'll discuss other functional language properties of JScript in a future post.

 

 

One can also construct anonymous functions at runtime with the Function constructor, though why you’d want to is beyond me.

 

 

print ( new Function("x", "return x \* 2;")(3) );

 

 

I recommend against constructing new functions at runtime based on strings, but that's a subject for a future post.

 

 

Interestingly enough, what this does internally is constructs the following script text, and compiles it:

 

 

function anonymous(x){return x \* 2;}

 

 

So in fact, **this actually does not compile up an anonymous function** -- it compiles up a non-anonymous function named "anonymous".   

 

 

The mind fairly boggles.

 

 

Second, there are the anonymous functions constructed when IE builds an event handler.   

 

 

ASIDE : In the source code for the script engine these things are called "scriptlets", which is a terrible, undescriptive, confusing name.  At one point there were three technologies all competing for the name "scriptlet".  The technology presently known as Windows Script Components was originally called "Scriptlets", which explains the name of the Wrox book "Instant Scriptlets" -- they published based on a beta released before the name was finalized.  We considered calling Windows Script Components "Scriptoids" and "Script Thingies", but fortunately cooler heads prevailed.

 

 

But I digress. When you have a button in IE

 

 

\<BLAH BLAH BLAH NAME="BUTTON1" ONCLICK="window.alert('Oooh, you clicked me\!');" \>

 

 

then the script engine creates a separate compilation scope for the form and compiles up this string:

 

 

function anonymous(){window.alert('Oooh, you clicked me\!');} 

 

 

It then passes the function object back as a dispatch pointer and IE assigns it to the button's onclick property.  Again, this is a non-anonymous function named "anonymous".

