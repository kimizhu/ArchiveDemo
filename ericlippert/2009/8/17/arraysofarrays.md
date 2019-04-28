# Arrays of arrays

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/17/2009 10:25:00 AM

-----

Most people understand that there’s a difference between a “rectangular” and a “ragged” two-dimensional array.

 

int\[,\] rectangle = {  
  {10, 20},  
  {30, 40},  
  {50, 60} };  
int\[\]\[\] ragged = {  
  new\[\] {10},  
  new\[\] {20, 30},  
  new\[\] {40, 50, 60} };

Here we have a two-dimensional array with six elements, arranged in three rows of two elements each. And we have a one-dimensional array with three elements, where each element is itself a one-dimensional array with one, two or three elements.

That’s all very straightforward. Where things get brain-destroying is when you try to make a ragged array of two-dimensional arrays.  Quick, don’t think, just answer.

 

int\[,\]\[\] crazy;

What is the type of crazy?

**Option one:** It’s a one-dimensional array, each element is a two-dimensional array of ints.  
**Option two:** It’s a two-dimensional array, each element is a one-dimensional array of ints.

OK, now that you have your snap answer, think about it carefully. Does your answer change?

I’m not going to tell you the answer just yet. Instead let’s explore the consequences of each possibility.

Consequence One

Surely the way you make any type into an array is to append \[\] to the type specification, right? But in option two, you stick the \[,\] into the *middle*.

Option two is weird. Option one is sensible. 

But wait. If \[,\]\[\] means "a 1-d of 2-d's", then *the order you read it off the page opposes the order you say it* -- it looks like two-d-thing-followed-by-one-d-thing, so why *shouldn't* it read "a 2-d of 1-d's"?

But then why does the "int" come first? By that logic it should come last.

Argh. Maybe option one isn't entirely sensible. Clearly something is not quite perfect with both options. Oh well. Let's move on.

Consequence Two

Now suppose that you wanted to obtain a value in that array, assuming that it had been initialized correctly to have plenty of elements everywhere we need them. How would you do it?

Suppose we’re in option one. It’s a one-d array. Therefore crazy\[10\] is a two-d array. Therefore crazy\[10\]\[1, 2\] is an int.

Suppose we’re in option two. It’s is a two-d array. Therefore crazy\[1,2\] is a one-d array. Therefore crazy\[1,2\]\[10\] is an int.

Option one is weird -- crazy is of type int\[,\]\[\] but you dereference it as \[10\]\[1,2\]\! Whereas option two is sensible; **the dereferencing syntax exactly matches the ordering of the type name syntax.**

Consequence Three

Now suppose you want to initialize the “outer” array but are going to fill in the “ragged” interior arrays later. You’ll just keep them set to null at first. What’s the appropriate syntax to initialize the outer array?

Suppose we’re in option one. It’s a one-d array. Therefore it should be initialized as crazy = new int\[,\]\[20\]; right?

Suppose we’re in option two. It’s a two-d array. Therefore it should be initialized as crazy = new int\[\]\[4,5\]; right?

Option two is weird. We said int\[,\]\[\] but initialized it as \[\]\[4,5\]. Option one is sensible.

What C\# actually does

It’s a mess. No matter which option we choose, something ends up not matching our intuition. Here’s what we actually chose in C\#.

First off: **option two is correct**. We force you to live with the weirdness entailed by Consequence One; **you do not actually make an element type into an array of that type by appending an array specifier**. You make it into an array type by *prepending the specifier to the list of existing array specifiers*. Crazy but true.

This prevents you from having to live with any weirdness from Consequence Two; in this option, the dereferencing happens with the same lexical structure as the declaration.

What about Consequence Three? This one is the real mind-bender. Neither choice I offered you is correct; apparently I’m a sneaky guy. The **correct** way to initialize such an array in C\# is:

 

crazy = new int\[4,5\]\[\];

This is very surprising to people\!

The design principle here is that **users expect the lexical structure of the brackets and commas to be consistent across declaration, initialization and dereferencing**. Option two is the best way to ensure that if declaration has the shape \[,\]\[\] then the initialization also has that shape, and so does the dereferencing.

That all said, **multidimensional ragged arrays are almost certainly a bad code smell**. Hopefully you will never, ever have to use your new knowledge of these rules in a production environment.

Life is much better if you can instead use generic collections. It is completely clear what List\<int\[,\]\> means – that’s a list of two-dimensional arrays. Whereas List\<int\>\[,\] means a two-d array of lists of int.

