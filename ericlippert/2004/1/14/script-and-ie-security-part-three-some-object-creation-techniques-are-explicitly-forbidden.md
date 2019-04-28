<div id="page">

# Script And IE Security Part Three: Some Object Creation Techniques Are Explicitly Forbidden

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/14/2004 11:41:00 AM

-----

<div id="content">

<span>A couple readers have commented that yes, my conjecture that people will continue to use the twentieth-century COM scripting interfaces for some time yet is correct.<span>  </span>(And my statement that the documentation on these has been less than 100% complete for the last seven years is also correct\!)<span>  </span>Thus, I'll probably follow up this series with articles on other low-level details on how the various script engine interfaces work.<span>  </span>I appreciate suggestions, so if anyone has ideas for things you'd like to see covered in more depth, let me know.<span>  </span>(Of course, you may want details on something I'm clueless about, in which case you'll have to learn to live with disappointment…) </span>

<span></span>

<span>But I digress.<span>  </span>There are three object creation scenarios which are automatically errors if either the "safe for untrusted data" or "use security manager" bits are set on the script engine.  I call them the **<span>DCOM</span>**, **<span>Moniker</span>** and **<span>Running Object</span>** scenarios: </span>

**<span class="underline"><span>DCOM</span></span>**

<span>DCOM, as you probably know, is just COM with longer wires.<span>  </span>(The details of invoking calls and moving data around become more complex as the wires get longer, but from the call site, the call looks pretty much the same.) The script engines provide syntax for objects to be created cross-machine via DCOM.<span>  </span>For example, in JScript Classic you can say</span>

<span>var someobj = new ActiveXObject(someprogid, someserver);</span>

<span>If the scripting security system is turned on (ie, either safe-for-untrusted-data or use-security-manager are set) then **<span>any attempt to create an object via DCOM automatically fails. </span>**There are no safe, partially-trusted DCOM scenarios.</span>

**<span class="underline"><span>Object Creation From Monikers</span></span>**

<span>A moniker is a "smart name".<span>  </span>That is, a name which has some code associated with it that can do something interesting.<span>  </span>A URL, for example, is a moniker which knows how to go out to the web and download the bits associated with the URL.<span>  </span>The script engines provide syntax for objects to be created via monikers:</span>

<span>var someobj = GetObject(somemoniker);</span>

<span>If the scripting security system is turned on (ie, either safe-for-untrusted-data or use-security-manager are set) then **<span>any attempt to create an object via a moniker automatically fails. </span>**There are no safe, partially-trusted scenarios where objects are loaded based on arbitrary monikers. </span>

**<span class="underline"><span>Object Creation Via the Running Object Table</span></span>**

<span>The script engines provide syntax for accessing already-created objects (usually in other processes, such as the Word object model):</span>

<span>var someobj = GetObject("", someprogid);</span>

<span>If the scripting security system is turned on (ie, either safe-for-untrusted-data or use-security-manager are set) then **<span>any attempt to create an object via the running object table automatically fails. </span>**There are no safe, partially-trusted scenarios where existing objects are accessed.  This could, among other things, break the IE domain security model whereby code from one web site is not allowed to access objects created by code from another web site.  It could also enable untrusted scripts to obtain access to powerful object models such as the Office object models.</span>

**<span class="underline"><span>Object Creation With Persistence but no Security Manager</span></span>**

<span>The script engines provide syntax for creating an object and persisting in state for that object:</span>

<span>var someobj = GetObject(somestatemoniker, someprogid);</span>

<span>If safe-for-untrusted-data is set and  use-security-manager is NOT set (that is, the script engine is not running in IE) then **<span>any attempt to create an object with persisted state automatically fails. </span>** Without access to the IE security manager the script engine has no way of knowing if the script is authorized to access the moniker.  The object might be a perfectly safe text box with its state persisted in from a file on the local file system, effectively stealing data from the file system and passing it to a trusted object scripted by an untrusted caller.</span>

<span>The case where there is a security manager is very complicated, so I will devote an entire entry to it later. </span>

<span></span>

<span>Tomorrow: how the script engines create objects in the case where there is no IE security manager available.<span>  </span>(I need to explain this first so that I can contrast it with how IE does it.)</span>

</div>

</div>

