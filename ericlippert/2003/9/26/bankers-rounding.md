<div id="page">

# Bankers' Rounding

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/26/2003 2:03:00 PM

-----

<div id="content">

<span> </span>

<span>A number of people have pointed out to me over the years that VBScript's </span><span>Round</span><span> function is a bit weird.  It seems like it should be pretty straightforward -- you pick the integer closest to the number you've got, end of story.  But what about, say, 1.5?  There are **<span>two</span>** closest integers.  Do you go up or down? </span>

<span></span>

<span>The </span><span>Round</span><span> function goes to the nearest integer, and *<span>if there are two nearest integers then it goes to the even one.</span>*  1.5 rounds to 2, 0.5 rounds to 0. </span>

<span></span>

<span>Why's that?  Why not just arbitrarily say that we always round down in this situation?  Why round down sometimes and up some other times?  There actually is a good reason\!  </span>

<span></span>

<span>This algorithm is called the ****<span>Bankers' Rounding</span>**** algorithm because, unsurprisingly, it's used by bankers.  Suppose a data source provides data which is often in exactly split quantities -- half dollars, half cents, half shares, whatever -- but they wish to provide rounded-off quantities.  Suppose further that a data consumer is going to derive summary statistics from the rounded data -- an average, say.  </span>

<span>Ideally when you are taking an average you want to take an average of the raw data with as much precision as you can get.  But in the real world we often have to take averages of data which has lost some precision.  In such a situation the Banker's Rounding algorithm produces better results because it does not bias half-quantities consistently down or consistently up.  It assumes that on average, an equal number of half-quantities will be rounded up as down, and the errors will cancel out.  </span>

<span>If you don't believe me, try it.  Generate a random list of numbers that end in 0.5, round them off, and average them.  You'll find that Bankers' Rounding gives you closer results to the real average than "always round down" averaging.</span>

<span></span>

<span>The </span><span>Round</span><span>, </span><span>CInt</span><span> and </span><span>CLng</span><span> functions in VBScript all use the Banker's Rounding algorithm.  </span>

<span></span>

<span>There are two other VBScript functions which turn floats into integers.  The </span><span>Int</span><span> function gives you the first integer *<span>less than or equal to its input</span>*, and the </span><span>Fix</span><span> function gives you the first integer *<span>closer to zero or equal to its input</span>*.  These functions do not round to the nearest integer at all, they simply truncate the fractional part.  </span>

<span></span>

<span>UPDATE: What about FormatNumber?  See [this post](http://blogs.msdn.com/ericlippert/archive/2003/09/26/53112.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/26/53112.aspx").</span>

</div>

</div>

