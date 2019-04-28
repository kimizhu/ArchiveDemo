# Script And IE Security Part Four: Creating Objects Without The IE Security Manager

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/15/2004 10:37:00 AM

-----

Suppose for a moment that a **non-IE** script host has asked the script engine to be safe for untrusted data but has not turned on the "use Security Manager" bit.  Suppose further the host passes in this data (from an untrustworthy and therefore potentially malicious source) to the JScript engine:

var fs = new ActiveXObject("Scripting.FileSystemObject");  
fs.DeleteFolder("c:\\windows", true);

The file system object is trusted, not inherently malicious but extremely dangerous in malicious hands.  The script security system is not sophisticated enough to distinguish between dangerous and safe methods of an object; rather it simply restricts creation of dangerous objects entirely.  (Again, this is a shortcoming which has been addressed by the CLR security system.)

This is the simplest possible object creation scenario: the IE security manager is not in use, the object has no persisted state, is not being accessed via DCOM and is a newly-created object. Let's dig into how exactly the script engine determines that this object is unsafe.  

First, the script engine uses the registry to determine which class id is associated with that progid, and then further which DLL is associated with that class.

Second, the DLL is loaded and the object's class factory is created.  Note that this runs code in the DLL. At this point the object is presumed to be trusted -- after all, it is installed on the user's machine.  If the user has made a poor trust decision and has installed a hostile object on their machine then at this point it is too late. Hostile code has already run (when DllMain was called by the loader.)  This calls out an important shortcoming of this model: code must be either fully trusted or not trusted at all.  The CLR security system has a “partial trust“ model whereby installed code can be given permission to run, but not to destroy your machine. 

Third, the actual object instance is created by the class factory and the newly-created object is QueryInterface'd for IObjectSafety.  If the object supports the safety interface then we set the "safe for scripting" bit so that the object knows that it needs to go into its "safe" mode.  If the object returns an error then we presume that the object cannot be made safe and refuse to create the object.

If the object does not support IObjectSafety then we use the component category manager to check the registry to see if CATID\_SafeForScripting is registered for the object.  This way producers of safe objects can mark their objects as safe without changing the source code, just the installation code.  (Objects which have this registry key set must always be safe; objects which implement IObjectSafety may have a "safe mode" and a "dangerous mode".)

If the object cannot tell the script engine that it is safe (either through the interface or the registry category) then the object is thrown away and an error is returned to the user.  ("Cannot create object") 

I'm AFK tomorrow, so we'll pick up next week with how object creation works differently when the IE security manager gets involved.

