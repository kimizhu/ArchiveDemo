<div id="page">

# Script In IE Security Part Six: Creating Objects With State

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/21/2004 11:00:00 AM

-----

<div id="content">

<span></span>

<span></span>

<span>Ideally the call to </span><span>QueryCustomPolicy</span><span> would take an argument representing the moniker for the persisted state so that this could be taken into account when the Security Manager made its decision whether or not to allow the persisted state to load.  Unfortunately, this scenario was overlooked when the Security Manager was designed, so we had to put the code into the script engines. </span>

<span></span>

<span>The script engine must determine whether the URL moniker for the persistent state comes from the same web site as the page. A page from [www.foo.com](http://www.foo.com/) should not be able to persist in state stored on [www.bar.com](http://www.bar.com/) (or from [file://c:blah.txt](file:///c:/blah.txt) <span> </span>\!) </span>

<span></span>

<span>The script engine does this by obtaining site identity strings for the URL and the current page and doing a bitwise comparison of them.  The site identity string for the page is obtained by calling </span><span>GetSecurityId</span><span> on the Internet Host Security Manager. The site identifier for the URL moniker is obtained slightly differently. We call </span><span>IInternetSecurityManager::GetSecurityId</span><span> (which returns the site id for an arbitrary URL) and not </span><span>IInternet**<span>Host</span>**SecurityManager::GetSecurityId</span><span>  (which gives the site id for the current page.)<span>  </span>People are often confused as to which is which\!</span>

<span>If the two site identity strings are not bit-for-bit identical then the object is thrown away and the script engine returns an error to the user.  If they are identical then the script engine uses the </span><span>IPersist</span><span> interfaces to load the state of the object and execution continues. </span>

<span>Object Creation Summary</span>

<span>**Preventing dangerous objects from being called by untrusted code is the main priority of the script engine security system.**  It does this by checking the object once, when it is created.  First it asks the security manager if the object can be trusted, then once the object is trusted it asks the security manager to verify that the object is safe to run.  Untrusted objects are never created, unsafe trusted objects are created but destroyed before any other code runs. </span>

<span></span>

<span>Next time, I'll talk a bit about some of the other security features in the script engines, like how they mitigate denial-of-service attacks.<span>  </span>I'm not sure what I'll do after that -- I might continue with my in-depth description of some of the script engine interfaces like </span><span>IDispatchEx</span><span> and </span><span>IActiveScript</span><span>, or I might get back to the topics I initially intended to cover, like how the CLR code security system improves on the script code security system.</span>

</div>

</div>

