# FYI: C\# and VB Closures are per-scope

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/6/2007 6:18:00 PM

-----

This post assumes that you understand [how closures are implemented in C\#.](http://blogs.msdn.com/oldnewthing/archive/2006/08/02/686456.aspx) They're implemented in [essentially the same way in the upcoming version of Visual Basic.](http://blogs.msdn.com/vbteam/archive/2007/05/25/closures-in-vb-part-3-scope.aspx)  As [Raymond](http://blogs.msdn.com/oldnewthing/default.aspx) and [Grant](http://blogs.msdn.com/grantri/default.aspx) point out in their various articles on the subject, the question of whether or not two instances of a delegate share a closed-over variable or have their own copy [depends on where the variable is in scope in relation to the delegate creation](http://blogs.msdn.com/oldnewthing/archive/2006/08/04/688527.aspx). I think that this issue is reasonably well-documented by these guys. [Lots has been written on this subject already](http://blogs.msdn.com/grantri/archive/category/3378.aspx); no need for me to recap it all here.

However, a related issue which I haven't seen anyone talk much about is what the consequences of having one closure per scope are. Though it makes the closure semantics conceptually easier to think about (and implement\!), it can lead to an unfortunate problem with garbage collection. Consider the following:

 

Func\<Cheap\> M(){  
  Cheap c = new Cheap();  
  Expensive e = new Expensive();  
  Func\<Expensive\> shortlived = ()=\>e;  
  Func\<Cheap\> longlived = ()=\>c;  
  // use shortlived  
  // use longlived  
  return longlived;  
} 

If the short-lived delegate does not survive past the end of the method, when is the expensive resource released?

The closure for the short-lived delegate owns the expensive resource. But since there is one closure per scope, both the short-lived and the long-lived delegates own a single closure. The closure cannot be collected until every delegate that owns it is dead. Therefore the expensive resource is not released until the long-lived delegate is released, even though the long-lived delegate does not reference the expensive resource\!

We could solve this problem in the compilers by coming up with a smarter mechanism for determining how to create closures, and perhaps some day we will, but it will not be in C\# 3.0. Until that day, if you are creating what you think are short-lived anonymous methods or lambdas or queries which close over expensive resources, you might want to explicitly finalize those resources when you know that you're done with them. It's easy to accidentally make the resource live longer than you think it does.

