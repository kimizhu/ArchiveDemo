<div id="page">

# Script And IE Security Part Two: Digging Deeper

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/13/2004 5:55:00 PM

-----

<div id="content">

<span></span>

<span>I want to continue my foray into the security semantics of the script engines this week, for a couple of reasons. </span>

<span></span>

<span>First of all, this information isn't really clearly documented anywhere outside of our internal documentation.<span>  Most of the interfaces are documented, but the logic behind them does not always shine through. </span>Though I hope that fewer and fewer developers use the hopelessly twentieth-century script engine technology as time goes by, I recognize that there will always be people who want to talk to the classic COM engines at a low level, so this stuff is for you guys. </span>

<span></span>

<span>Second, the script engines are a good case study in how to develop secure software.<span>  </span>We have made many mistakes over the years and found many flaws, and clearly the CLR security model is orders of magnitude better, but the basic principles and techniques we used are sound.<span>  </span>(One hopes that the CLR security team started with our security model and tried to make it better\!) </span>

<span></span>

<span>The next few posts are going to be considerably more low-level than my usual posts, but we'll be back to the higher-level stuff eventually. </span>

<span></span>

<span>Note that, as usual, I will be using words like "trusted" and "safe" in specific, technical ways.<span>  </span>See my [previous posting](http://blogs.msdn.com/ericlippert/archive/2003/09/25/53097.aspx) on that subject for details. </span>

<span></span>

<span>Enough chit-chat\!<span>  </span>Let's go\! </span>

<span></span>

<span>Script Security -- Why Bother?</span>

<span></span>

<span>The VBScript and JScript engines are arguably the most important DLLs shipped with Windows from a security-vulnerability perspective.  The primary purpose of the script engines is to take **<span>arbitrary source code</span>** from **<span>potentially hostile</span>**, **<span>untrusted</span>** web sites, compile it **<span>and execute it</span>**.  The programs created can create arbitrary objects.  It is hard to think of a more dangerous activity available to a larger number of customers and hence available to a larger number of attackers.  Therefore **<span>the script engines must have a carefully designed and implemented security system to allow harmless code to run while restricting hostile, dangerous code.</span>**</span>

<span>I will concentrate on the "script in IE" scenario, though I might also explore scripting in the context of other hosts like ASP, WSH, etc, in future posts.  The IE scenario is the most complicated scenario with the greatest likelihood of encountering hostile code.</span>

<span>Can We Just Turn It Off?</span>

<span>Prima fascia it seems like a sensible thing to say would be "if scripting in IE is so incredibly dangerous then why is it on?  Why don't we just turn it off by default?"</span><span> </span>

<span></span>

<span>"Off by default" is the mantra at Microsoft, and it makes a lot of sense. The attitude used to be something like this: "Running the HTTP service is cheap on non-server machines, and the user might want their machine to be a server.<span>  </span>We want to make it easy on our customers to quickly and easily set up web servers, so let's turn it on by default."<span>  </span>Unfortunately, when there is a flaw in the web server, that means that EVERY machine is now vulnerable whether it is actually a web server or not.<span>  </span>The new mantra is "Turning the HTTP service on is easy if you're an administrator, so let's ship with it off by default.<span>  </span>That way if there is a security hole, it only affects a small number of customers, not everyone.  Defense in depth -- attackers can't attack code that isn't even running." </span>

<span>If that is such a good strategy then why ship IE with scripting turned on by default?<span>  </span>The simple fact is that millions of web pages (including most of the most popular pages) require scripting in order to render correctly.  If we tell millions of users that their choices are "turn scripting off and have an **<span>unusable web experience”</span>** vs. “turn scripting on but it might be insecure" then *<span>they will turn it back on every time.</span>*  Turning scripting off by default would just force millions of users to turn it back on, and then they'd be in the same boat as they are now -- only more irked and confused.  </span>

<span></span>

<span>There is always a balance between security and usability.<span>  </span>We have to find the right balance.<span>  </span>The script engines have to be secure by default, but still usable. </span>

<span></span>

<span>Trust and Safety</span>

<span>The security system is designed to make the user's trust decisions easy and automatic through good default settings and easily-configured custom settings.  Various mechanisms like the digital signature system for ActiveX controls are designed to provide evidence for the user to use when making trust decisions.</span>

<span>A trusted control is not necessarily safe ; rather **<span>a trusted control is assumed to honestly report whether it is safe or dangerous</span>**.  A trusted control is also assumed to be non-malicious -- we assume that trusted code will not format your hard disk when you call <span>QueryInterface </span>on it. If the user trusts a malicious control then the user has somehow made a poor trust decision (by, say, turning off the security system, or downloading unsigned ActiveX controls from random web sites.)  </span>

<span>A trusted control is also assumed to be *<span>correct</span>*. A non-malicious and seemingly safe control might in fact be dangerous if it has, say, a buffer overrun bug.</span>

<span>The aim of the scripting security system is therefore to make untrusted scripts safe by restricting their abilities to a safe sandbox. It does this by (among other things) **<span>only creating trusted ActiveX objects which claim to be safe.</span>**</span>

<span>Note that generally speaking, any object installed on the user's machine is fully trusted. The scripting security system assumes that by installing it the user has chosen to trust the object.  **<span>The scripting security system does nothing to verify the trustworthiness of an object</span>**; that is the responsibility of the user, IE's digital signature verification mechanisms, etc.</span>

<span>(This trust model has some obvious shortcomings that have been addressed in the CLR security model.  I hope to delve into those in more detail in a future entry.)</span>

<span>Turning Security On</span>

<span>That's enough introductory material. let's start getting into the low-level interfaces.</span>

<span>The primary control surface for security in the script engines is the </span><span>IObjectSafety </span><span>interface, which has two methods:</span>

<span>interface IObjectSafety : IUnknown </span><span>{  
</span><span>       HRESULT GetInterfaceSafetyOptions(  
</span><span>              \[in\]  REFIID  riid,                // Interface that we want options for  
</span><span>              \[out\] DWORD \* pdwSupportedOptions, // Options meaningful on this interface  
</span><span>              \[out\] DWORD \* pdwEnabledOptions);  // current option values on this interface  
</span><span>       HRESULT SetInterfaceSafetyOptions(  
</span><span>              \[in\]  REFIID  riid,                // Interface to set options for  
</span><span>              \[in\]  DWORD   dwOptionSetMask,     // Options to change  
</span><span>              \[in\]  DWORD   dwEnabledOptions);   // New option values  
</span><span>}</span>

<span>Note that these methods get/set security bits on the particular *<span>interfaces</span>* implemented by an object.  An object could in theory have an interface </span><span>ISafe </span><span>and an interface </span><span>IDangerous </span><span>with completely different security semantics.  As a practical matter however, the script engines ignore the </span><span>riid</span><span> parameter. It makes no sense to have the script engine have one kind of security behaviour when called from </span><span>IActiveScript </span><span>and another when called from </span><span>IActiveScriptParse</span><span>\!</span>

<span>There are four flags supported by </span><span>IObjectSafety</span><span>:</span>

<span>INTERFACESAFE\_FOR\_UNTRUSTED\_CALLER 0x00000001   // Caller of interface may be untrusted  
</span><span>INTERFACESAFE\_FOR\_UNTRUSTED\_DATA  </span><span><span> </span></span><span>0x00000002   // Data passed into interface may be untrusted  
</span><span>INTERFACE\_USES\_DISPEX             </span><span><span> </span></span><span>0x00000004   // Object knows to use IDispatchEx  
</span><span>INTERFACE\_USES\_SECURITY\_MANAGER   </span><span><span> </span></span><span>0x00000008   // Object knows to use IInternetHostSecurityManager</span><span> </span>

<span>**Understanding the distinction between the first two is crucial. ** </span>

****

****

<span>An object which claims to support the "safe for untrusted callers" bit is saying "**<span>it does not matter what methods you call on me, I cannot possibly do anything harmful</span>**."  Essentially the object is saying that its capabilities are so weak that no matter how hostile the caller, nothing bad will happen.  The "Scripting.Dictionary" object is an example of such an object -- no matter what, it cannot do anything more than store a list of (key, value) pairs.  It cannot format your hard disk or send your private data to [www.evilbadguys.com](http://www.evilbadguys.com).  </span>

<span>An object which supports this bit is also known as a "**<span>safe for scripting</span>**" object -- it may be safely called from an untrusted script.</span>

<span>An object which claims to support the "safe for untrusted data" bit is making a somewhat different claim.  There might be some way of calling this object that causes some dangerous operation, so the calls have to be from a trusted caller. But once that condition is satisfied, **<span>the arguments to those calls can be data from an untrusted source, such as a web site</span>**.  </span>

**<span>The script engines are NOT safe for untrusted callers but are safe for untrusted data.</span>**<span>  If an untrusted piece of code could get hold of a script engine's <span>IUnknown </span>interface then that untrusted code could do anything -- like, say, turning off the script engine's security via the above interface\! But a script engine is safe for untrusted data.  A trusted caller (like Internet Explorer) can make calls to the script engine and pass in untrusted data (like code downloaded from the Internet) and rest assured that nothing bad will happen.</span>

<span>The latter two flags, "use </span><span>IDispatchEx</span><span> " and "use Security Manager" are only relevant on script engines. These bits are used by Internet Explorer to tell the script engine to use some very IE-specific security semantics which we will get into in great detail later.  The script engines support both these flags.</span>

<span>Note that it is nonsensical to set the "use Security Manager" flag and not set the "safe for untrusted data" flag on a script engine.  Various code paths in the script engine implicitly assume that if "safe for untrusted data" is NOT set then the engine is NOT in "safe mode".</span><span> </span>

<span></span>

<span>Stay tuned.<span>  </span>Next time, how the script engines decide whether an object should be created or not.</span>

</div>

</div>

