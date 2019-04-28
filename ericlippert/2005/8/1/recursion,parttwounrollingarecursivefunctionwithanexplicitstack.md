# Recursion, Part Two: Unrolling a Recursive Function With an Explicit Stack

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/1/2005 6:00:00 AM

-----

That recursive solution is pretty cool, to be sure. But there is one big problem with it.  Consider this Jscript program that puts a few more nodes into our tree from last time: function tree(value, left, right)  
{  
  this.value = value;  
  this.left = left;  
  this.right = right;  
}  
for (var i = 1 ; i \< 1500 ; ++i)  
  mytree = new tree(i, mytree, null)  
print(treeDepth(mytree)); This is not a very well-behaved binary tree – it’s actually mostly a linked list, hardly a tree at all.  Recursing down 1500 branches to get to the bottom blows the stack.  Each time you enter a JScript function, that pushes a new JScript "activation" frame on the JScript stack. But the JScript stack is actually allocated off of the system heap, so this isn't the problem. Unfortunately, doing so also pushes a new copy of the interpreter on the **system** stack, which consumes a few hundred bytes of stack space.  Since by default the system stack only has a million bytes available, this quickly consumes the entire system stack. **Recursive functions are sometimes the most elegant solution, but not always the most practical.** How are we going to solve this problem? One way to do it is to not get into the situation in the first place. There are various algorithms for guarateeing that when you construct a binary tree that it is a "balanced" binary tree. That is, there are no long, deep paths. A *perfectly* balanced binary tree with 1500 nodes in it is guaranteed to require no more than 11 recursive steps, and doubling the number of nodes only adds on one more step.  A perfect tree with a million nodes requires no more than 20 recursive steps, so odds are pretty good that you'll never blow the stack if you know you have a balanced tree.  There are algorithms such as the famous red-black algorithm that guarantee that a binary tree is always within a factor of two of being perfectly balanced, so maybe a million-node tree is 40 deep, which seems doable. Another approach is to keep track of how deep you are on the stack and bail out: function treeDepth(curtree, stackdepth)  
{  
  if (curtree == null)  
    return 0;  
  if (stackdepth \> 1000)  
    throw "tree is too deep, bailing out\!";  
  return 1 + Math.max(  
    treeDepth(curtree.left, stackdepth + 1),  
    treeDepth(curtree.right, stackdepth + 1));  
} But that's pretty hacked up. What if you're wrong about how much stack you have left? Maybe 1000 is too deep. What if you can't guarantee that the tree is balanced?  What if you want to have an algorithm that works for *any* binary tree without danger of blowing the stack or getting a goofy exception? Some languages, such as Scheme, have what’s called “tail recursion”, where they detect when they can optimize away pushing another frame on the stack, and therefore do not have this problem. But JScript does not have this optimization, and even if it did, this tree algorithm is a poor candidate for tail recursion. Tail recursion works really well when the recursive step is last, but this algorithm has *two* recursive steps, so one of them is going to be not last, obviously. (Though as we'll see in a few episodes, there are things we can do about that.) Hmm. I just said that the JScript stack is actually allocated off of the system heap -- the interpreter simulates a stack for the script engines rather than re-using the system stack. We can do the same thing with our recursive program: **make the stack explicit rather than implicit.** That's going to be complicated. Think about what has to happen every time you call a function in JScript. Somehow the interpreter has to remember a whole bunch of things, like:

  - values of local variables in the current activation frame -- variables and arguments in the callee's frame must not change the variables in the caller's frame.
  - where to pick up execution when the callee returns
  - what to do with the return value of the callee
  - values of "anonymous locals" -- for example, when you have x = 1 + 1 + foo(); somehow the engine has to remember that there's a 2 sitting there in temporary storage waiting to be added when foo returns.

Let's take our recursive depth algorithm and refactor it a little bit so that the temporary variables are spelled out and each recursion happens on its own line: function treeDepth(curtree)  
{  
  if (curtree == null)  
    return 0;  
  var leftdepth = treeDepth(curtree.left);  
  var rightdepth = treeDepth(curtree.right);  
  return 1 + Math.max(leftdepth, rightdepth);  
} For each recursive step we'll need a frame that keeps track of what the three local variables are.  Before each recursion we need to remember where we were, so that when we start executing, we know whether to pick up on the "var leftdepth..." line or "var rightdepth..." line or the "return 1 + ..." line.  We'll remember that by keeping track of whether leftdepth and/or rightdepth are initialized on the frame.  When we complete a calculation we'll pop off the current frame and fill in the "calling" frame's variables with the appropriate "return value" so that our execution loop knows where to go next. function treeDepth(curtree)  
{  
  if (curtree == null)  
    return 0;  
  var frame = {  
    curtree:    curtree,  
    leftdepth:  null,  
    rightdepth: null};  
  var stack = new Array();  
  stack.push(frame);  
  while(true)  
  {  
    var curframe = stack\[stack.length-1\];  
    if (curframe.curtree == null)  
    {  
      stack.pop();  
      curframe = stack\[stack.length-1\];  
      if (curframe.leftdepth == null)  
        curframe.leftdepth = 0;  
      else  
        curframe.rightdepth = 0;  
    }   
    if (curframe.leftdepth == null)  
    {  
      curframe = {  
        curtree:    curframe.curtree.left,  
        leftdepth:  null,  
        rightdepth: null};  
      stack.push(curframe);  
    }  
    else if (curframe.rightdepth == null)  
    {  
      curframe = {  
        curtree:    curframe.curtree.right,  
        leftdepth:  null,  
        rightdepth: null};  
      stack.push(curframe);  
    }  
    else  
    {  
      var depth = 1 + Math.max(curframe.leftdepth, curframe.rightdepth);  
      stack.pop();  
      if (stack.length == 0)  
        return depth;  
      curframe = stack\[stack.length-1\];  
      if (curframe.leftdepth == null)  
        curframe.leftdepth = depth;  
      else  
        curframe.rightdepth = depth;  
    }  
  }  
} Since there is no recursion here at all now, there's no danger of running out of stack. But good heavens, that's **a freakin' mess compared to the three-line recursive solution.** This is probably more trouble than it's worth. Next time we'll look at another explicit-stack based approach that is a whole lot cleaner than this dog's breakfast. (Eric is on vacation; this message was prerecorded.)

