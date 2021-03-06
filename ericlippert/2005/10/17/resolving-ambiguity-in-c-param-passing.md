<div id="page">

# Resolving ambiguity in C\# param passing

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/17/2005 12:53:00 PM

-----

<div id="content">

Here's a question I got recently about parameter arrays in C\#. Suppose you are designing a method and you know that it is going to take some small number of values, but you're not sure how many.  For example, consider what I call the "madlib" formatting functions: Console.WriteLine("{0} love my big sphinx of {1}", bird, rock); You don't know *until runtime* how many arguments that thing is going to take, so you can use a parameter array to take a variable number of arguments. Now suppose that the *caller* of that method also doesn't know how many arguments there are going to be until runtime. Perhaps the arguments are going to be based on some kind of user input and their number will not be known. Since we've already got this handy form that takes a parameter array, we can just pass an array of the appropriate type in as the final parameter and the language will pass the parameter array right along. Unfortunately, this leads to a problem when the parameter array is of type object\[\]. Suppose you have a method which takes a parameter array of objects, and you want to pass in an array of objects as the one and only argument? That is, suppose myobjectarr is {10, null, "abc"}.  Foo.MyFunc(myobjectarr); will be the same as Foo.MyFunc(10, null, "abc"), but an array of objects is a perfectly good object -- what if you want to pass that as the only argument rather than doing the expansion? Fortunately there's an easy way around this. You give the compiler a little hint that says "even though this thing is an object array, treat it as though it is not convertible to one". Foo.MyFunc(**(object)**myobjectarr); does the trick. See section 10.5.1.4 of the C\# specification for more details.  

 

</div>

</div>

