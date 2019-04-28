<div id="page">

# Script And IE Security Part Four: Creating Objects Without The IE Security Manager

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/15/2004 10:37:00 AM

-----

<div id="content">

<span>Suppose for a moment that a **<span>non-IE</span>** script host has asked the script engine to be safe for untrusted data but has not turned on the "use Security Manager" bit.  Suppose further the host passes in this data (from an untrustworthy and therefore potentially malicious source) to the JScript engine:</span>

<span>var fs = new ActiveXObject("Scripting.FileSystemObject");  
</span><span>fs.DeleteFolder("c:\\windows", true);</span>

<span>The file system object is trusted, not inherently malicious but extremely dangerous in malicious hands.  The script security system is not sophisticated enough to distinguish between dangerous and safe methods of an object; rather it simply restricts creation of dangerous objects entirely.  (Again, this is a shortcoming which has been addressed by the CLR security system.)</span>

<span>This is the simplest possible object creation scenario: the IE security manager is not in use, the object has no persisted state, is not being accessed via DCOM and is a newly-created object. L</span><span>et's dig into how exactly the script engine determines that this object is unsafe.  </span>

<span></span>

<span>First, the script engine uses the registry to determine which class id is associated with that progid, and then further which DLL is associated with that class.</span>

<span>Second, the DLL is loaded and the object's class factory is created.  Note that this runs code in the DLL. At this point the object is presumed to be trusted -- after all, it is installed on the user's machine.<span>  </span>If the user has made a poor trust decision and has installed a hostile object on their machine then at this point it is too late. Hostile code has already run (when DllMain was called by the loader.)  This calls out an important shortcoming of this model: code must be either fully trusted or not trusted at all.  The CLR security system has a “partial trust“ model whereby installed code can be given permission to run, but not to destroy your machine. </span>

<span></span>

<span>Third, the actual object instance is created by the class factory and the newly-created object is </span><span>QueryInterface</span><span>'d for </span><span>IObjectSafety</span><span>.  If the object supports the safety interface then we set the "safe for scripting" bit so that the object knows that it needs to go into its "safe" mode.<span>  </span>If the object returns an error then we presume that the object cannot be made safe and refuse to create the object.</span>

<span>If the object does not support </span><span>IObjectSafety</span><span> then we use the component category manager to check the registry to see if </span><span>CATID\_SafeForScripting</span><span> is registered for the object.  This way producers of safe objects can mark their objects as safe without changing the source code, just the installation code.  (Objects which have this registry key set must always be safe; objects which implement </span><span>IObjectSafety</span><span> may have a "safe mode" and a "dangerous mode".)</span>

<span>If the object cannot tell the script engine that it is safe (either through the interface or the registry category) then the object is thrown away and an error is returned to the user.  ("Cannot create object") </span>

<span></span>

<span></span>

<span>I'm AFK tomorrow, so we'll pick up next week with how object creation works differently when the IE security manager gets involved.</span>

</div>

</div>

