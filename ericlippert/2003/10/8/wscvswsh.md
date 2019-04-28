# WSC vs WSH

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2003 4:57:00 PM

-----

 

 

Following up on this morning's entry, a reader asked me why Windows Script Components don't have access to the WScript object.  "*it IS running in an instance of WSH isnt it?"* 

 

 

No, it isn't.  That's a common misperception.  Let me clear it up.

 

 

Basically the whole point of scripting is to make it easy to write programs.  Two very common kinds of programs are **executables** and **components**.  I'm sure that you understand the difference between the two, but just let me emphasize that executables are "standalone" -- they have a well defined startup, they run for a while, and then they shut down.  Components are libraries of useful functionality packaged up as objects.  These objects cannot live on their own -- they have to be created by an executable or another component, and they live as long as the owner wants them to.

 

 

We provide two ways to write "executables in script" -- Windows Script Host files (.VBS, .JS, .WSF) and HTML Applications (.HTA).  When you run one of these, the host application starts up, loads the script, runs it, and then shuts down.   

 

 

We also provide a way to write components in script -- the aptly named Windows Script Components.

 

 

When you create a WSC, its no different from creating any other in-process COM object.  The fact that the object is implemented in script is irrelevant.    

 

 

People get WSCs and WSF's confused because of their similar names and syntaxes.  But logically, WSCs and WSH are completely separate entities.   

 

 

Note though that under the covers, quite a bit of the WSF processing code is actually implemented in ScrObj.DLL, the WSC engine, which itself is consumed as a component by the WSH executable.  We saw no reason to implement two identical XML parsers, two debugger interfaces, etc, when we had one sitting right there in a DLL already.

