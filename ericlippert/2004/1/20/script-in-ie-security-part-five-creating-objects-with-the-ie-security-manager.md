<div id="page">

# Script In IE Security Part Five: Creating Objects With The IE Security Manager

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/20/2004 1:15:00 PM

-----

<div id="content">

<span>I spent the weekend falling down Whistler and Blackcomb mountains with slippery boards attached to each foot.<span>  </span>I'm getting *better* at falling at least -- in all my wipeouts I managed to keep my skis on, which was a big improvement over last year's multiple “yard sale” experience.<span>  </span>Now I'm working on that fall, backwards somersault, get on skis, continue down mountain trick.<span>  </span>(I suppose that's the ski equivalent of capsizing a Laser and righting it without getting wet, which I'm already pretty good at.) </span>

<span></span>

<span>But, enough chit-chat.<span>  </span>Last week I was discussing object creation.<span>  </span>Let's pick that up, and next time I'll discuss how objects with state are created. </span>

<span></span>

<span>Script In IE Security Part Five: Creating Objects With The IE Security Manager</span>

<span>Object creation becomes somewhat more complicated if the IE security manager is available.</span>

<span>First, as before, the script engine gets the class id and code location from the registry. </span>

<span>Second, the script engine obtains the Internet Host Security Manager.  This is done by calling </span><span>QueryService</span><span> on the script engine's site (ie, the script engine host application) for </span><span>SID\_SInternetHostSecurityManager</span><span>.  Once the </span><span>IInternetHostSecurityManager</span><span> is obtained the script engine obtains policy bits representing Internet Explorer's position on the classid by calling </span><span>ProcessUrlAction(URLACTION\_ACTIVEX\_RUN)</span><span>.  If the policy does not have the </span><span>URLPOLICY\_ALLOW</span><span> bit set or if the call to </span><span>ProcessUrlAction</span><span> failed for any reason then the object is never created and the script engine reports an error to the user.</span>

<span>The purpose of this check on the classid is so that IE can have a list of "known to be untrustworthy" classids and prevent them from running at all.  Also, IE can check its security configuration at this point -- if ActiveX objects are turned off altogether it can tell the script engine that nothing is safe to create.</span>

<span>Once the security manager approves the object for creation the script engine does the same thing as before, as far as creating a class factory and object instance goes.  At this point the object is trusted and its code has run. However, the subsequent safety check is somewhat different from the previous case.  If the "use the Security Manager" bit is set then **<span>the script engines do not do any safety check on the objects; they assume that the Security Manager will do that</span>**. The script engine does that by passing the object's </span><span>IUnknown</span><span> and classid  to </span><span>QueryCustomPolicy(GUID\_CUSTOM\_CONFIRMOBJECTSAFETY).</span><span>  (If the script engine is planning on loading persisted state it also specifies the </span><span>CONFIRMSAFETYACTION\_LOADOBJECT</span><span> flag -- more on persisted state tomorrow.)</span>

<span>The Security Manager then does all the appropriate </span><span>IObjectSafety</span><span> work, including setting the "be safe for untrusted data" bit on the object if the script engine is planning to load in persisted state.  If any of this fails then the Security Manager reports that back to the script engine, which throws away the object and reports an error to the user.  </span>

<span></span>

<span>Depending on the IE security settings, the browser may create a dialog box to ask the user to make a trust decision right there.  Note that if this happens then the browser asks whether the user trusts the *<span>page author</span>*, not whether the user trusts the *<span>individual controls</span>*. This is an interesting distinction, which I may come back to later.  (I'd also like to talk about the pros and cons of security dialog boxes -- see Peter Torr's blog today for our comments on security dialog boxes and Outlook.)</span>

<span>At this point if there is no state to load we are done. The Security Manager has claimed that the object is safe to create and safe to run.  If there is persisted state then we have more work to do...</span>

</div>

</div>

