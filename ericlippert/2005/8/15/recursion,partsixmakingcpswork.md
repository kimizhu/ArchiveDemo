# Recursion, Part Six: Making CPS Work

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/15/2005 6:00:00 AM

-----

JScript doesn't support CPS natively but we can write another dispatch engine that makes it work. There's only ever one active continuation, so lets have a new rule:  JScript CPS functions are allowed to return, but the last thing that they do must be to tell our dispatch engine what the continuation was. To keep things simple, we'll also say **that every CPS-style function takes exactly one argument**. Of course, that can be an object that contains multiple fields should the function require multiple arguments. Let's go through our CPS treeDepth program and replace all the continuation calls with calls that tell the runtime engine what continuation to use next.  We'll then return normally and let the runtime engine call the continuation for us. function treeDepth(args)  
{  
  if (args.curtree == null)  
    cont(args.afterDepth, 0);  
  else  
  {  
    function afterLeft(leftDepth)  
    {  
      function afterRight(rightDepth)  
      {  
        cont(args.afterDepth, 1+Math.max(leftDepth, rightDepth));     
      }  
      cont(treeDepth, {curtree: args.curtree.right, afterDepth: afterRight});  
    }  
    cont(treeDepth, {curtree: args.curtree.left, afterDepth: afterLeft});  
  }  
} The dispatch engine is very simple: var continuation = null;  
function cont(newfunc, newargs)  
{  
  continuation = { func: newfunc, args : newargs };  
} function run()  
{  
  while(continuation \!= null)  
  {  
    var curfunc = continuation.func;  
    var curargs = continuation.args;  
    continuation = null;  
    curfunc(curargs);  
  }  
} No stack at all this time. Just a single global variable saying what to call next. If there is nothing to call next, we're done. To determine the depth of a tree we simply tell the continuation engine what the next thing to do is, and start it running: cont(treeDepth, {curtree: mytree, afterDepth: print});  
run(); and hey presto, we've just implemented a recursive function with no stack that prints the depth of an arbitrary binary tree. Nice trick\! Of course, **really what we've done is we've just hidden the call stack in another harder-to-see data structure. Where is it?** (Hint: draw some diagrams that show what all the closure name table bindings are during the recursion.) That's all I managed to get done here before my vacation. When I get back, maybe I'll *pop the stack* and pick up where we left off, or maybe I'll *continue* with something completely different and forget all about my current state.  (Or maybe I'm actually interrupt driven, which seems most likely.) Who knows? Whatever happens though, it'll probably be another **fabulous adventure in coding**. \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Further reading: Don Box on closures and CPS in C\# 2.0: <http://pluralsight.com/blogs/dbox/archive/2005/04/27/7780.aspx> (Eric is on vacation; this message was pre-recorded.)

