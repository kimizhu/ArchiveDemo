# What are closures?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2003 6:32:00 PM

-----

[JScript, as I noted yesterday, is a **functional language**](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53021.aspx). That doesn't mean that it works particularly well (though I hope it does) but rather that it treats functions as first-class objects. Functions can be passed around and assigned to variables just as strings or integers can be.

A reader commented yesterday that "closures are your friends". Well, actually there are important situations where closures are NOT your friends\! Let's talk a bit about those. First off, **what's a closure**?

Consider the following (contrived and silly, but pedagocially clear) code:

function AddFive(x) {  
  return x + 5;  
}

function AddTen(x) {  
  return x + 10;  
}

var MyFunc;  
if (whatever)  
  MyFunc = AddFive;  
else  
  MyFunc = AddTen;  
print(MyFunc(123)); // Either 133 or 128.

Here we have a typical functional scenario. We're deciding which function to call based on some runtime test. Now, one could imagine that you'd want to generalize this notion of an "adder function", and you would not want to have to write dozens and dozens of adders. What we can do is create an adder factory:

function AdderFactory(y) {  
  return function(x){return x + y;}  
}

var MyFunc;  
if (whatever)  
  MyFunc = AdderFactory(5);  
else  
  MyFunc = AdderFactory(10);  
print(MyFunc(123)); // Either 133 or 128.

The anonymous inner function **remembers** what the value of y was when it was returned, even though y has gone away by the time the inner function is called\! We say that **the inner function is closed over the containing scope**, or for short, that the inner function is a **closure**.

This is an extremely powerful functional language feature, but it is important to not misuse it. There are ways to cause memory-leak-like situations using closures. Here's an example:

\<body\>  
\<div class='menu-bar' id='myMenu'\>\</div\>  
\<script language='javascript'\>  
var menu = document.getElementById('myMenu');  
AttachEvent(menu);  
function AttachEvent(element) {  
  element.attachEvent( "onmouseover", mouseHandler);  
  function mouseHandler(){ /\* whatever \*/ }  
}  
\</script\>\</body\>

Someone has, for whatever reason, nested the handler inside the attacher. This means that the handler is closed over the scope of the caller; the handler keeps around a reference to element which is equal to menu, which is that div. But the div has a reference to the handler.

That's a circular reference.

Now, the [JScript garbage collector is a mark-and-sweep GC](http://blogs.msdn.com/ericlippert/archive/2003/09/17/53038.aspx)so you'd think that it would be immune to circular references. But the IE div isn't a JScript object; it is not in the JScript GC, so the circular reference between the div and the handler will not be broken until the browser completely tears down the div.

Which never happens.

This page used to say that IE tears down the div when the page is navigated away, but it turns out that that's not right.  Though IE did briefly do that, the application compatibility lab discovered that there were actually web pages that broke when those semantics were implemented.  (No, I don't know the details.) The IE team considers breaking existing web pages that used to work to be way, way worse than leaking a little memory here and there, so they've decided to take the hit and leak the memory in this case.

**Don't use closures unless you really need closure semantics. In most cases, non-nested functions are the right way to go.**

