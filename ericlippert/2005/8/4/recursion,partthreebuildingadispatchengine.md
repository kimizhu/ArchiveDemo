# Recursion, Part Three: Building a Dispatch Engine

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/4/2005 6:00:00 AM

-----

There's a particular technique that I like to use to solve problems -- it doesn't always work, but when it does, it can produce programs of surprising elegance and power. The technique is this: if you have a specific problem to solve, solve a more general problem, and then your problem is just a special case. It's like that joke someone mentioned during my series on high-dimensional arithmetic -- it's easy to imagine a 9-dimensional space, just imagine an n-dimensional space and set n=9. That sounds like more work, doesn't it?  Often it is, but paradoxically, sometimes it's less work. Our solution last time for de-recursivizing a three-line recursive algorithm was an unreadable mess in part because the explicit stack logic was all mixed up with the bits of the function actually doing work. Rather than messing around trying to implement JScript's call logic in JScript itself, complete with frames and return values and activation objects, let's solve a more general problem. Let's build a little dispatch engine, and build some functions that use that dispatch engine to solve problems. Solving tree depth will then just be a special case. Our dispatch engine will have two stacks: the functions-I-still-need-to-call stack, and the values-to-pass-to-them stack.  But to keep it simple, the functions themselves will be responsible for taking the values on and off the value stack. With those design characteristics, our "dispatch engine" becomes very, very simple.  It just pops a function off the function stack and calls it. When there are no more functions on the stack, we're done. while(functionstack.length \!= 0)  
  functionstack.pop()(); We'll need to do a little stack magic to make it all work, but we can do it. We'll call the functions D, R and M: D:  
**action**: either computes the depth of a null node or pushes a program that "recursively" computes the depth of a node.  
**value stack precondition**: value stack top contains tree node  
**value stack postcondition**: if tree node on stack is null, replace it with zero. Otherwise push right and left trees  
**function stack postcondition**: if tree node is not null, push MDRD onto function stack R:  
**action**: reorganizes the value stack so that the depth of the right tree can be determined  
**value stack precondition**: top of stack is left tree depth, followed by tree node  
**value stack postcondition**: top of stack is tree node, followed by left tree depth  
  
M:  
**action**: takes the max + 1 of two numbers  
**value stack precondition**: two numbers on top of stack  
**value stack postcondition**: one number on top of stack Does that make sense?  Let's think about how that would play out for a tree with root B, left child A, right child C.  I'll use X to represent an empty (null) tree. We put D on the function stack and B on the value stack and start the engine: functions values notes  
D         B      B is not null, so we push its children on the value stack and MDRD on the function stack  
MDRD      CA     Again, D is the top of the function stack, so we do it again, this time on A  
MDRMDRD   CXX    Now D is at the top of the function stack and we have a null. Replace it with zero.  
MDRMDR    CX0    Swap the top two  
MDRMD     C0X    Replace null with zero  
MDRM      C00    M pops both numbers, finds the larger, adds one, pushes it  
MDR       C1     Swap the top two, replace the tree with its right child  
MD        1C     and so on...  
MMDRD     1XX  
MMDR      1X0  
MMD       10X  
MM        100  
M         11  
          2 And if we've done it right, when we're done the value stack contains the depth of the tree that we started with, and the function stack is empty. Looks promising. Let's write the code. var functionstack = new Array();  
var valuestack = new Array(); function treeDepth()  
{  
  var curtree = valuestack.pop();  
  if (curtree == null)  
    valuestack.push(0);  
  else  
  {  
    functionstack.push(max);  
    functionstack.push(treeDepth);  
    functionstack.push(reorder);  
    functionstack.push(treeDepth);  
    valuestack.push(curtree.right);   
    valuestack.push(curtree.left);  
  }  
} function reorder()  
{  
  var leftdepth = valuestack.pop();  
  var righttree = valuestack.pop();  
  valuestack.push(leftdepth);  
  valuestack.push(righttree);  
} function max()  
{  
  valuestack.push(1 + Math.max(valuestack.pop(), valuestack.pop()));  
} functionstack.push(treeDepth);  
valuestack.push(mytree);  
while(functionstack.length \!= 0)  
  functionstack.pop()();  
print(valuestack.pop()); Each function is a lot cleaner now that the execution management has been pushed to an external engine. But we've paid a pretty high price in that it's now non-obvious that this program *actually works* on all trees. I mean, it sure looks like it works, but we haven't proven it. But at least its not directly recursive, so we're pretty sure that we don't have to worry about running out of system stack. This time we attacked the problem of eliminating recursion by getting rid of the notion of explicit frames on the stack. Next time we'll take a very different tack and try to eliminate stacks altogether.  
(Eric is on vacation; this message was pre-recorded.)

