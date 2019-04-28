# Running Me Ragged

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/22/2003 9:34:00 PM

-----

 

 

A reader asked **"Why are there two types of multidimentional arrays? What is the difference between the ****(x)(y) and (x,y) notation?"**

 

 

Good question.  There are two kinds of multidimensional arrays, called "**rectangular**" and "**ragged**".  A rectangular array is, well, rectangular.  You say

 

 

DIm MyArrary(3,2)

 

 

and you get an array with indices:

 

 

(0,0)  (0,1)  (0,2)

(1,0)  (1,1)  (1,2)

(2,0)  (2,1)  (2,2)

(3,0)  (3,1)  (3,2)

 

 

which makes a nice rectangle.  A three-dimensional array makes a rectangular prism, and so on up into the higher dimensions.

 

 

Now, as I mentioned earlier, JScript does not have multidimensional arrays.  A clever trick to simulate multidimensional arrays is to make an array of arrays:

 

 

var x = new Array(

  new Array(1, 2, 3), 

  new Array(4, 5), 

  new Array(6, 7, 8, 9));

 

 

And so dereferencing the outer array gives you the inner array, which can then be dereferenced itself:

 

 

print(x\[2\]\[0\]); // 6

 

 

But you notice something about the indices if we write them out as before:

 

 

\[0\]\[0\]   \[0\]\[1\]   \[0\]\[2\]

\[1\]\[0\]   \[1\]\[1\]

\[2\]\[0\]   \[2\]\[1\]   \[2\]\[2\]   \[2\]\[3\]

 

 

The indices make a ragged pattern, not a straight rectangular pattern.

 

 

You can have ragged higher dimensional arrays as well, though allocating all the sub-arrays gets to be a royal pain. 

 

 

There are often times when you want ragged arrays even in a language that supports rectangular multi-dimensional arrays, so VBScript supports both.  If you say

 

 

MyArray(2,3) 

 

 

then you are talking to a rectangular two-dimensional array.  If you say

 

 

MyArray(2)(3)

 

 

then you are talking to a one dimensional array that contains another one dimensional array.

