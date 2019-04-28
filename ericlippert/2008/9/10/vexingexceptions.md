# Vexing exceptions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/10/2008 3:28:00 PM

-----

Writing good error handling code is hard in any language, whether you have exception handling or not. When I'm thinking about what exception handling I need to implement in a given program, I first classify every exception I might catch into one of four buckets which I label **fatal**, **boneheaded**, **vexing** and **exogenous**.

**Fatal** exceptions are *not your fault*, you *cannot prevent them*, and you *cannot sensibly clean up from them*. They almost always happen because the process is deeply diseased and is about to be put out of its misery. Out of memory, thread aborted, and so on. There is absolutely no point in catching these because nothing your puny user code can do will fix the problem. Just let your "finally" blocks run and hope for the best. (Or, if you're really worried, fail fast and do not let the finally blocks run; at this point, they might just make things worse. But that's a topic for another day.)

**Boneheaded** exceptions are *your own darn fault*, you *could have prevented them* and therefore *they are bugs in your code*. You should not catch them; doing so is hiding a bug in your code. Rather, you should write your code so that the exception cannot possibly happen in the first place, and therefore does not need to be caught. That argument is null, that typecast is bad, that index is out of range, you're trying to divide by zero – these are all problems that you could have prevented very easily in the first place, so prevent the mess in the first place rather than trying to clean it up.

**Vexing** exceptions are the result of *unfortunate design decisions*. Vexing exceptions are thrown in a completely non-exceptional circumstance, and therefore must be caught and handled all the time.

The classic example of a vexing exception is Int32.Parse, which throws if you give it a string that cannot be parsed as an integer. But the 99% use case for this method is transforming strings input by the user, which could be any old thing, and therefore it is in no way *exceptional* for the parse to fail. Worse, there is no way for the caller to determine ahead of time whether their argument is bad *without implementing the entire method themselves*, in which case they wouldn't need to be calling it in the first place.

This unfortunate design decision was so vexing that of course the frameworks team implemented TryParse shortly thereafter which does the right thing.

You have to catch vexing exceptions, but doing so is vexing.

Try to never write a library yourself that throws a vexing exception.

And finally, **exogenous** exceptions appear to be somewhat like vexing exceptions except that they are not the result of unfortunate design choices. Rather, they are the result of untidy external realities impinging upon your beautiful, crisp program logic. Consider this pseudo-C\# code, for example:

 

try  
{  
  using ( File f = OpenFile(filename, ForReading) )  
  {  
    // Blah blah blah  
  }  
}  
catch (FileNotFoundException)  
{  
  // Handle filename not found  
}

Can you eliminate the try-catch? 

 

if (\!FileExists(filename))  
  // Handle filename not found  
else  
  using ( File f = ...

This isn't the same program. There is now a "race condition". Some other process could have deleted, locked, moved or changed the permissions of the file between the FileExists and the OpenFile.

Can we be more sophisticated? What if we lock the file? That doesn't help. The media might have been removed from the drive, the network might have gone down…

You’ve got to catch an exogenous exception because it always could happen no matter how hard you try to avoid it; it’s an exogenous condition outside of your control.

So, to sum up:

• Don’t catch fatal exceptions; nothing you can do about them anyway, and trying to generally makes it worse.  
• Fix your code so that it never triggers a boneheaded exception – an "index out of range" exception should never happen in production code.  
• Avoid vexing exceptions whenever possible by calling the “Try” versions of those vexing methods that throw in non-exceptional circumstances. If you cannot avoid calling a vexing method, catch its vexing exceptions.  
• Always handle exceptions that indicate unexpected exogenous conditions; generally it is not worthwhile or practical to anticipate every possible failure. Just try the operation and be prepared to handle the exception.

