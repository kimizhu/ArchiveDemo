# Bankers' Rounding

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/26/2003 2:03:00 PM

-----

 

A number of people have pointed out to me over the years that VBScript's Round function is a bit weird.  It seems like it should be pretty straightforward -- you pick the integer closest to the number you've got, end of story.  But what about, say, 1.5?  There are **two** closest integers.  Do you go up or down? 

The Round function goes to the nearest integer, and *if there are two nearest integers then it goes to the even one.*  1.5 rounds to 2, 0.5 rounds to 0. 

Why's that?  Why not just arbitrarily say that we always round down in this situation?  Why round down sometimes and up some other times?  There actually is a good reason\!  

This algorithm is called the ****Bankers' Rounding**** algorithm because, unsurprisingly, it's used by bankers.  Suppose a data source provides data which is often in exactly split quantities -- half dollars, half cents, half shares, whatever -- but they wish to provide rounded-off quantities.  Suppose further that a data consumer is going to derive summary statistics from the rounded data -- an average, say.  

Ideally when you are taking an average you want to take an average of the raw data with as much precision as you can get.  But in the real world we often have to take averages of data which has lost some precision.  In such a situation the Banker's Rounding algorithm produces better results because it does not bias half-quantities consistently down or consistently up.  It assumes that on average, an equal number of half-quantities will be rounded up as down, and the errors will cancel out.  

If you don't believe me, try it.  Generate a random list of numbers that end in 0.5, round them off, and average them.  You'll find that Bankers' Rounding gives you closer results to the real average than "always round down" averaging.

The Round, CInt and CLng functions in VBScript all use the Banker's Rounding algorithm.  

There are two other VBScript functions which turn floats into integers.  The Int function gives you the first integer *less than or equal to its input*, and the Fix function gives you the first integer *closer to zero or equal to its input*.  These functions do not round to the nearest integer at all, they simply truncate the fractional part.  

UPDATE: What about FormatNumber?  See [this post](http://blogs.msdn.com/ericlippert/archive/2003/09/26/53112.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/26/53112.aspx").

