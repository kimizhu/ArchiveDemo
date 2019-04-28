# Recursion, Part Five: More on CPS

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/11/2005 6:00:00 AM

-----

Suppose we wanted to write this by-now-familiar little function in continuation passing style: function treeDepth(curtree)  
{  
  if (curtree == null)  
    return 0;  
  else  
  {  
    var leftDepth = treeDepth(curtree.left);  
    var rightDepth = treeDepth(curtree.right);  
    return 1 + Math.max(leftDepth, rightDepth);  
  }  
} Let's start by getting rid of the returns, since there are no returns in CPS. Rather, we "return" by passing the return value to the continuation. Remember, the continuation is "what are we supposed to do after this function is done?" function treeDepth(curtree, afterDepth)  
{  
  if (curtree == null)  
    afterDepth(0);  
  else  
  {  
    //UNDONE: var leftDepth = treeDepth(curtree.left);  
    //UNDONE: var rightDepth = treeDepth(curtree.right);  
    afterDepth(1 + Math.max(leftDepth, rightDepth));  
  }  
} We'll presume that the addition and maximum guys are not written in CPS, just to keep this simple. OK, this is a start, but now we need to fix up those recursive calls.  Consider the second recursive call.  What is its continuation?  That is, what needs to happen after the right tree depth is determined?  Two things. First, the maximum of left and right must be calculated and incremented.  Second, we've promised our caller that when we're done, we'll call its continuation.  So let's write a closure that does those two things. function treeDepth(curtree, afterDepth)  
{  
  if (curtree == null)  
    afterDepth(0);  
  else  
  {  
    function afterRight(rightDepth)  
    {  
      afterDepth(1 + Math.max(leftDepth, rightDepth));  
    }  
    // UNDONE: var leftDepth = treeDepth(curtree.left);    
    treeDepth(curtree.right, afterRight);  
  }  
} We're making progress. What do we need to do after we determine the depth of the left tree?  Pass the left depth to a continuation that recurses on the right tree to determine the right depth, does the max, and calls the afterDepth continuation. function treeDepth(curtree, afterDepth)  
{  
  if (curtree == null)  
    afterDepth(0);  
  else  
  {  
    function afterLeft(leftDepth)  
    {  
      function afterRight(rightDepth)  
      {  
        afterDepth(1 + Math.max(leftDepth, rightDepth));  
      }  
      treeDepth(curtree.right, afterRight);  
    }  
    treeDepth(curtree.left, afterLeft);  
  }  
} And we're done.  We've got a CPS version of treeDepth which never returns and therefore we never need to keep track of what is on the stack, because we're never coming back. Of course, like I said last time, in reality JScript does **not** detect that we're in this situation and optimize away the stack frames.  This program still pushes interpreters on the system stack and still runs out of stack space if you have a 1500-deep branch in your tree. But we can fake it out.  Next time, we'll finish up with a CPS program that really doesn't run out of stack space, even in JScript. (Eric is on vacation; this message was pre-recorded.)

