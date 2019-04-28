# Script And IE Security Part Seven: Other Stuff

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/22/2004 10:20:00 AM

-----

The only JScript functions which change their semantics when the security system is on are the ActiveXObject constructor and the GetObject method which we have already covered.  VBScript has only four methods which are restricted or changed when the security system is on.  CreateObject works exactly like the ActiveXObject constructor, GetObject works just like JScript's GetObject.  That leaves InputBox and MsgBox. The two dialog box functions have some interesting security features:

If the VBScript engine is in "safe mode" then it prefixes dialog box titles with "VBScript" to prevent possible spoofing.  It also turns off any "help" button on the dialog box -- help files can be on the local disk, and we are paranoid about allowing access to the local disk. If there were, say, a buffer overrun error in the help system... that might be bad.   

Also the contents of the dialogs are restricted so that the dialog cannot be made larger than the screen and the dialog position is clamped so that it is created on the screen.  This last point is to prevent the irritating denial-of-service attack where a hostile caller creates a message box moved to just outside the visible screen. If we allowed message boxes to be created off the screen then the user would have no idea why the browser stopped working.  The modal dialog cannot be seen, so users would be quite perplexed. 

As I'm sure you've noticed, the script engines make no attempt to restrict trivial, irritating denial-of-service attacks such as creating a large number of modal dialogs one after the other, creating lots of irritating pop-up browser windows, and so on.  (I personally would like there to be a big red button on the desktop labeled "MR. WORF\! SHIELDS UP\!"  that would automatically stop all scripts from running, shut down the internet connection, and so on.  Perhaps I'll implement it one of these days.)

There are, of course, other DoS issues. What if someone writes a script that has an infinite loop? The script engines in IE provide some mechanisms to abort long-running scripts -- these could be buggy scripts with infinite loops or deliberate attempts to consume processor cycles.  However, these mechanisms are relatively weak.  All these mechanisms are based on aborting the script processing but the abort signal is not processed until the beginning of the first script statement following the signal.

This therefore eliminates run-of-the-mill attacks such as sitting in a tight infinite loop. It does not eliminate attacks where the script engine is made to call into some third-party object that takes a long time to return.

The mechanisms include the IActiveScriptSitePoll interface, whereby the script engine calls the host back every few statements and asks whether the script should be continued, and the IActiveScript::InterruptScriptThread method which asynchronously sets a bit on the script engine.  If the bit is on then the script is shut down when the beginning of the next statement is reached. There is no safe way to shut down a script engine in a truly asynchronous manner -- and please don't just terminate the thread\!  (See <http://weblogs.asp.net/oldnewthing/archive/2003/12/09/55988.aspx> for details.)

No attempt is made to throttle the amount of memory allocated by an untrusted script.  This can lead to both a memory DoS and a time DoS because as the number of allocations monitored by the JScript garbage collector increases, the time spent in the garbage collector looking for blocks to free increases as well.

**Safe-for-scripting objects are responsible for ensuring that they cannot be leveraged by a DoS attack.** For example, an object which allowed the untrusted script to dump data to a local printer ought not to be marked as safe -- a hostile page could use up all the user's paper and toner. 

Next time I'll wrap up this in-depth look at the script engine security features with an article on how IDispatchEx is used to enable behaviour similar to the CLR security system's stack walks

