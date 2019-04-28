# It's the most wonderful time of the year

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/26/2009 8:01:00 AM

-----

Here's a little holiday cheer for you all. Or, at least for you all in Commonwealth countries.

 

static object M\<T\>(T t) where T : struct  
{  
  return t;  
}

int ii = 10;  
int? jj = 20;  
object xx = ii;  
object yy = jj;  
System.ValueType zz = ii;  
IComparable aa = ii;  
System.Enum bb = MidpointRounding.ToEven;  
object cc = M(ii);

I hope you're having a festive holiday season. Happy Boxing Day, and we'll see you in the New Year for more fabulous adventures\!

\[Eric is on vacation; this posting is pre-recorded.\]

