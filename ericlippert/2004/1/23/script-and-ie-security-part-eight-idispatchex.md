<div id="page">

# Script And IE Security Part Eight: IDispatchEx

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/23/2004 1:20:00 PM

-----

<div id="content">

<span>Finally, we get to the last unexplained bit on the </span><span>IObjectSafety</span><span> interface.<span>  </span>What's the "use dispex" flag for? </span>

<span></span>

<span>I'll go into the rationale for the </span><span>IDispatchEx</span><span> interface in another entry.<span>  </span>As far as its security impact goes, we had to solve certain scenarios involving Microsoft's implementation of the Java VM.  The specific details of these scenarios are lost in the mists of time -- as you might imagine, the people who worked on the Microsoft Java VM have been doing other things for some time now, and I was never an expert on the interactions between the script engines, IE and the VM.<span>  </span>But clearly they had to deal with the same fundamental issue that the CLR stack-walking code deals with today: when the runtime executes potentially dangerous code it must be able to get security information about the callers to determine whether the entire call stack is sufficiently trusted. </span>

<span>To understand the stack walking mechanism the first thing to understand is how </span><span>IServiceProvider::QueryService</span><span> differs from </span><span>IUnknown::QueryInterface</span><span>.  </span><span>QueryInterface</span><span> asks an object "are you an </span><span>IFoo</span><span>?"  </span><span>QueryService</span><span> says to an object "Please find me an object that implements </span><span>IFoo</span><span> and provides service </span><span>SBar</span><span>."  </span>

<span>If the object queried does not happen to provide service </span><span>SBar</span><span> then it may pass that request along to some other object that it thinks might provide that service.  It in turn may pass the request along again.  Obviously it is important to ensure that this spate of buck-passing does not form a loop, otherwise an unbounded recursion could result (which would quickly consume all available stack and then crash.)  For this reason most service providers defer "up the stack" in some sense, so that the algorithm terminates.</span>

<span>The </span><span>IDispatchEx::InvokeEx</span><span> method takes a service provider as an argument. **<span>The callee may then query the service provider in an attempt to get information about the caller.</span>**  When the script engine is **<span>called</span>** via </span><span>IDispatchEx::InvokeEx</span><span> it saves off a pointer to the caller's service provider.  It also may obtain a service provider from the script site (i.e., IE).</span>

<span>When the script engine then **<span>calls</span>** an object via </span><span>IDispatchEx::Invoke</span><span> it provides a new service provider of its own.  This service provider provides two services relevant to this discussion:  </span><span>SID\_GetCaller</span><span> and </span><span>SID\_GetScriptSite</span><span>.  If asked for a different service then the script engine's service provider passes the buck to the script's caller (or to the site if there is no caller provider available.)</span>

<span>This mechanism thereby allows the callee to do three things:</span>

<span>1)</span><span>      </span><span>The callee may obtain services from IE. For instance, if the callee asks for </span><span>SID\_SInternetSecurityManager</span>*<span> </span>*<span>then eventually the call will be proxied to the script site. </span>

<span>2)</span><span>      </span><span>The callee may obtain a pointer to the site directly via </span><span>SID\_GetScriptSite</span><span>.</span>

<span>3)</span><span>      </span><span>The callee may obtain pointers to service providers for each script engine and other </span><span>IDispatchEx</span><span>-aware objects on the call stack.</span>

<span>In theory, this mechanism allows callees to obtain information about callers at every call site on the stack.<span>  </span>In practice, as far as I'm aware, this mechanism is pretty much solely used to get the security manager off the call site. </span>

<div>

<span></span>

</div>

<span></span>

<span>That pretty much wraps up this series on the low-level security semantics of the script engines themselves.<span>  </span>I hope it was useful.<span>  </span>I've got a bunch of ideas for series to do next, including: </span>

<span></span>

  - <span>similar low-level security details about WSH -- how code signing works, for example.<span>  </span>Or, a reader recently asked about Remote WSH, which is not very well known.</span>
  - <span>a detailed discussion of the motivation behind the IActiveScript and IDispatchEx interfaces</span>
  - <span>a discussion of how the CLR security system extends and improves the ActiveX security system </span>

<span></span>

<span>I'll try to find some time to review my notes over the weekend, and we'll get started sometime next week.</span>

</div>

</div>

