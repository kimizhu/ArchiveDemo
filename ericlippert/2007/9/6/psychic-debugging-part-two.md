<div id="page">

# Psychic Debugging, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/6/2007 11:31:00 AM

-----

<div id="content">

<div class="mine">

A number of readers have the [mysterious fifth sense](http://blogs.msdn.com/ericlippert/archive/2004/08/20/i-have-a-mysterious-fifth-sense.aspx) which gives them the ability to deduce that the <span class="code">GetBars</span> method from [yesterday's post](http://blogs.msdn.com/ericlippert/archive/2007/09/05/psychic-debugging-part-one.aspx) contains a <span class="code">yield return</span> and is therefore an iterator. Remember, as the standard states (in section 10.14.4):

<span class="spec"> </span>

\[...\] execution of the code in the iterator block occurs when the enumerator object's MoveNext method is invoked.

Since the test program does not invoke <span class="code">MoveNext</span>, the check for <span class="code">null</span> is never executed, and therefore the exception is never thrown.

Since most of the interesting new sequence operators available in C\# 3.0 are implemented with iterators it probably will be increasingly important for developers to understand a bit more about how iterators work behind the scenes. I may do some blog posts on that over the next little while.

</div>

</div>

</div>

