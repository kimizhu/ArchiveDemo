# Breadth is sometimes better than depth

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/27/2004 10:58:00 AM

-----

A while back the

[Scripting Guys](http://blogs.msdn.com/gstemp) blogged about [using recursion to list all the files in a directory](http://blogs.msdn.com/gstemp/archive/2004/08/10/212113.aspx) and all its subdirectories. Something that you'll notice about this very standard solution to this problem is that it is "depth first". That is, it lists the files like this: c:\\xyz.txt  
c:\\foo\\bbb.txt  
c:\\foo\\abc\\def.txt  
c:\\foo\\bar\\baz.txt  
c:\\foo\\bar\\blah.txt  
c:\\qaz\\zaq.txt  
c:\\qaz\\lll\\ggg.txt It picks a branch and follows it down the tree as far as it can, listing directories as it goes, retreats back up as **little as possible**, picks another branch to go down, goes as far down it as possible, and so on, until the entire tree has been explored. You can see why this is called "depth first". You don't *need* to use recursion to do this; recursion is just an elegant way to solve the problem. The essence of the algorithm is that information about the next node to process is stored in a **stack**. The algorithm is simple: pop the stack, process the directory, push all the subdirectories onto the stack, repeat until the stack is empty. Recursion just lets you do this algorithm without making the stack explicit; the stack is the call stack and *doing the recursive call pushes a new frame on the call stack for you*. We can easily do this without recursion by making the stack explicit: var FSO = new ActiveXObject("Scripting.FileSystemObject");  
var stack = new Array();  
stack.push(FSO.GetFolder("."));  
while(stack.length \> 0)  
{  
  var folder = stack.pop();  
  for (var enumtor = new Enumerator(folder.Subfolders) ; \!enumtor.atEnd(); enumtor.moveNext())  
  {  
    try  
    {  
      stack.push(enumtor.item());  
    }  
    catch(e){}  
  }  
  for (enumtor = new Enumerator(folder.Files) ; \!enumtor.atEnd(); enumtor.moveNext())  
  {  
    try  
    {  
      WScript.Echo(enumtor.item().Path);  
    }  
    catch(e){}  
  }  
} Notice that I've wrapped the file manipulation code in try-catches. As I've [mentioned before](http://blogs.msdn.com/ericlippert/archive/2004/08/25/220373.aspx), in this script if there is a problem reading a file -- like, I don't own it and therefore I get a security violation -- I want to skip the problem and continue on. I do not want to write robust error handling for this trivial one-off script. The other day I had a problem -- a few weeks earlier I had written a little test program and stuck it in a directory somewhere very near the top of my directory hierarchy, but I couldn't remember where exactly. I already had the depth-first program above, and could easily modify it to add a regular expression search of each file looking for a pattern. But there are some **very** deep directories on that drive where I knew there would be no matches, but would take a lot of time to search. I could have written code to omit those directories, but there was another way to solve this problem: since I knew that the file was going to be somewhere shallow, don't do a depth-first search in the first place. Do a **breadth-first** search. That is, search all the files in the top level, then all the files that are one deep, then all the files that are two deep, then three deep, and so on. The nice thing is that it's basically the same algorithm, just with a minor change. Instead of doing a last-in-first-out stack, we'll just do a first-in-first-out queue and magically get a breadth-first traversal: var FSO = new ActiveXObject("Scripting.FileSystemObject");  
var queue = new Array();  
queue\[queue.length\] = FSO.GetFolder(".");  
var counter = 0;  
while(counter \< queue.length)  
{  
  var folder = queue\[counter\];  
  for (var enumtor = new Enumerator(folder.Subfolders) ; \!enumtor.atEnd(); enumtor.moveNext())  
  {  
    try  
    {  
      queue\[queue.length\] = enumtor.item();  
    }  
    catch(e) {}  
  }  
  for (enumtor = new Enumerator(folder.Files) ; \!enumtor.atEnd(); enumtor.moveNext())  
  {  
    try  
    {  
      WScript.Echo(enumtor.item().Path);  
    }  
    catch(e){}  
  }  
  queue\[counter\] = null;  
  counter++;  
} This then puts out the files in the order c:\\xyz.txt  
c:\\qaz\\zaq.txt  
c:\\foo\\bbb.txt  
c:\\qaz\\lll\\ggg.txt  
c:\\foo\\abc\\def.txt  
c:\\foo\\bar\\baz.txt  
c:\\foo\\bar\\blah.txt going from shallowest to deepest gradually rather than continually diving way down and then coming back up.

