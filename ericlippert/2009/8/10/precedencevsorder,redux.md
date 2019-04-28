# Precedence vs order, redux

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/10/2009 10:06:00 AM

-----

Once more I'm revisting [the myth that order of evaluation has any relationship to operator precedence in C\#](http://blogs.msdn.com/ericlippert/archive/2008/05/23/precedence-vs-associativity-vs-order.aspx). Here's a version of this myth that I hear every now and then. Suppose you've got a field arr that is an array of ints, and some local variables index and value:

 

int index = 0;  
int value = this.arr\[index++\];

When all is said and done, value will contain the value that was in this.arr\[0\], and index will be 1, right?

Right. Now, the myth. I often hear this explained as "because the ++ comes after the variable, the increment **happens after** the array is dereferenced."

Wrong\! "After" implies **a relationship based on a sequence of events in time**, and the *logical* sequence of events is extremely well-defined. The sequence of events for this program fragment is:

1\) store zero in "index"  
2\) fetch a reference to this.arr and remember the result  
3\) fetch the value in "index" and remember the result  
4\) add one to the result of step 3 and remember the result  
5\) store the result of step 4 in "index"  
6\) look up the value in the reference from step 2 at the index from step 3 and remember the result  
7\) store the result of step 6 in "value"

I emphasize that the logical sequence is well-defined because the compiler, jitter and processor are all allowed to change the actual order of events for optimization purposes, subject to the restriction that the **optimized result must be indistinguishable from the required result in single-threaded scenarios**. Reorderings *are* sometimes observable in multithreaded scenarios. Analyzing the consequences of that fact is insanely complicated. I might blog about those issues at some point if I feel brave. But for normal scenarios, you should assume that the ordering of events in time precisely follows the rules laid out in the specification.

In fact, you can demonstrate that the increment happens before the indexing with this little self-referential gem:

 

int\[\] arr = {0};  
int value = arr\[arr\[0\]++\];

What happens? First we fetch arr\[0\], which is 0, and remember that. Then we increment arr\[0\], so arr\[0\] becomes 1. Then we fetch the value of arr\[0\] because we remember the 0, and we get 1.  If we had done the increment *after* the (outer) indexing then the result would be zero because the increment would not happen until after the value had been fetched.

\-- Eric is on vacation; this posting was prerecorded --

