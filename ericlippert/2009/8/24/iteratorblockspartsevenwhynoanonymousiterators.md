# Iterator Blocks Part Seven: Why no anonymous iterators?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/24/2009 9:23:00 AM

-----

This annotation to a comment in part five I think deserves to be promoted to a post of its own.

**Why do we disallow anonymous iterators?** I would love to have anonymous iterator blocks.  I want to say something like:

IEnumerable\<int\> twoints = ()=\>{ yield return x; yield return x\*10; };  
foreach(int i in twoints) ... 

It would be totally awesome to be able to build yourself a little sequence generator in-place that closed over local variables. The reason why not is straightforward: the benefits don't outweigh the costs. The awesomeness of making sequence generators in-place is actually pretty small in the grand scheme of things and nominal methods do the job well enough in most scenarios. So the benefits are not that compelling.

The costs are large. Iterator rewriting is the most complicated transformation in the compiler, and anonymous method rewriting is the second most complicated. Anonymous methods can be inside other anonymous methods, and anonymous methods can be inside iterator blocks. Therefore, what we do is first we rewrite all anonymous methods so that they become methods of a closure class. This is the second-last thing the compiler does before emitting IL for a method. Once that step is done, the iterator rewriter can assume that there are no anonymous methods in the iterator block; they've all be rewritten already. Therefore the iterator rewriter can just concentrate on rewriting the iterator, without worrying that there might be an unrealized anonymous method in there.

Also, iterator blocks never "nest", unlike anonymous methods. The iterator rewriter can assume that all iterator blocks are "top level".

If anonymous methods are allowed to contain iterator blocks, then both those assumptions go out the window. You can have an iterator block that contains an anonymous method that contains an anonymous method that contains an iterator block that contains an anonymous method, and... yuck. Now we have to write a rewriting pass that can handle nested iterator blocks and nested anonymous methods at the same time, merging our two most complicated algorithms into one far more complicated algorithm. It would be really hard to design, implement, and test. We are smart enough to do so, I'm sure. We've got a smart team here. But we don't want to take on that large burden for a "nice to have but not necessary" feature.

