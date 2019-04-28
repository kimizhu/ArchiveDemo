# Script And IE Security Part Eight: IDispatchEx

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/23/2004 1:20:00 PM

-----

Finally, we get to the last unexplained bit on the IObjectSafety interface.  What's the "use dispex" flag for? 

I'll go into the rationale for the IDispatchEx interface in another entry.  As far as its security impact goes, we had to solve certain scenarios involving Microsoft's implementation of the Java VM.  The specific details of these scenarios are lost in the mists of time -- as you might imagine, the people who worked on the Microsoft Java VM have been doing other things for some time now, and I was never an expert on the interactions between the script engines, IE and the VM.  But clearly they had to deal with the same fundamental issue that the CLR stack-walking code deals with today: when the runtime executes potentially dangerous code it must be able to get security information about the callers to determine whether the entire call stack is sufficiently trusted. 

To understand the stack walking mechanism the first thing to understand is how IServiceProvider::QueryService differs from IUnknown::QueryInterface.  QueryInterface asks an object "are you an IFoo?"  QueryService says to an object "Please find me an object that implements IFoo and provides service SBar."  

If the object queried does not happen to provide service SBar then it may pass that request along to some other object that it thinks might provide that service.  It in turn may pass the request along again.  Obviously it is important to ensure that this spate of buck-passing does not form a loop, otherwise an unbounded recursion could result (which would quickly consume all available stack and then crash.)  For this reason most service providers defer "up the stack" in some sense, so that the algorithm terminates.

The IDispatchEx::InvokeEx method takes a service provider as an argument. **The callee may then query the service provider in an attempt to get information about the caller.**  When the script engine is **called** via IDispatchEx::InvokeEx it saves off a pointer to the caller's service provider.  It also may obtain a service provider from the script site (i.e., IE).

When the script engine then **calls** an object via IDispatchEx::Invoke it provides a new service provider of its own.  This service provider provides two services relevant to this discussion:  SID\_GetCaller and SID\_GetScriptSite.  If asked for a different service then the script engine's service provider passes the buck to the script's caller (or to the site if there is no caller provider available.)

This mechanism thereby allows the callee to do three things:

1\)      The callee may obtain services from IE. For instance, if the callee asks for SID\_SInternetSecurityManager* *then eventually the call will be proxied to the script site. 

2\)      The callee may obtain a pointer to the site directly via SID\_GetScriptSite.

3\)      The callee may obtain pointers to service providers for each script engine and other IDispatchEx-aware objects on the call stack.

In theory, this mechanism allows callees to obtain information about callers at every call site on the stack.  In practice, as far as I'm aware, this mechanism is pretty much solely used to get the security manager off the call site. 

That pretty much wraps up this series on the low-level security semantics of the script engines themselves.  I hope it was useful.  I've got a bunch of ideas for series to do next, including: 

  - similar low-level security details about WSH -- how code signing works, for example.  Or, a reader recently asked about Remote WSH, which is not very well known.

  - a detailed discussion of the motivation behind the IActiveScript and IDispatchEx interfaces

  - a discussion of how the CLR security system extends and improves the ActiveX security system 

I'll try to find some time to review my notes over the weekend, and we'll get started sometime next week.

