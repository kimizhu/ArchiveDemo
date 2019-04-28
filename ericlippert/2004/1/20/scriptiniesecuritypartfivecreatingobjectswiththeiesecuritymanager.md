# Script In IE Security Part Five: Creating Objects With The IE Security Manager

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/20/2004 1:15:00 PM

-----

I spent the weekend falling down Whistler and Blackcomb mountains with slippery boards attached to each foot.  I'm getting *better* at falling at least -- in all my wipeouts I managed to keep my skis on, which was a big improvement over last year's multiple “yard sale” experience.  Now I'm working on that fall, backwards somersault, get on skis, continue down mountain trick.  (I suppose that's the ski equivalent of capsizing a Laser and righting it without getting wet, which I'm already pretty good at.) 

But, enough chit-chat.  Last week I was discussing object creation.  Let's pick that up, and next time I'll discuss how objects with state are created. 

Script In IE Security Part Five: Creating Objects With The IE Security Manager

Object creation becomes somewhat more complicated if the IE security manager is available.

First, as before, the script engine gets the class id and code location from the registry. 

Second, the script engine obtains the Internet Host Security Manager.  This is done by calling QueryService on the script engine's site (ie, the script engine host application) for SID\_SInternetHostSecurityManager.  Once the IInternetHostSecurityManager is obtained the script engine obtains policy bits representing Internet Explorer's position on the classid by calling ProcessUrlAction(URLACTION\_ACTIVEX\_RUN).  If the policy does not have the URLPOLICY\_ALLOW bit set or if the call to ProcessUrlAction failed for any reason then the object is never created and the script engine reports an error to the user.

The purpose of this check on the classid is so that IE can have a list of "known to be untrustworthy" classids and prevent them from running at all.  Also, IE can check its security configuration at this point -- if ActiveX objects are turned off altogether it can tell the script engine that nothing is safe to create.

Once the security manager approves the object for creation the script engine does the same thing as before, as far as creating a class factory and object instance goes.  At this point the object is trusted and its code has run. However, the subsequent safety check is somewhat different from the previous case.  If the "use the Security Manager" bit is set then **the script engines do not do any safety check on the objects; they assume that the Security Manager will do that**. The script engine does that by passing the object's IUnknown and classid  to QueryCustomPolicy(GUID\_CUSTOM\_CONFIRMOBJECTSAFETY).  (If the script engine is planning on loading persisted state it also specifies the CONFIRMSAFETYACTION\_LOADOBJECT flag -- more on persisted state tomorrow.)

The Security Manager then does all the appropriate IObjectSafety work, including setting the "be safe for untrusted data" bit on the object if the script engine is planning to load in persisted state.  If any of this fails then the Security Manager reports that back to the script engine, which throws away the object and reports an error to the user.  

Depending on the IE security settings, the browser may create a dialog box to ask the user to make a trust decision right there.  Note that if this happens then the browser asks whether the user trusts the *page author*, not whether the user trusts the *individual controls*. This is an interesting distinction, which I may come back to later.  (I'd also like to talk about the pros and cons of security dialog boxes -- see Peter Torr's blog today for our comments on security dialog boxes and Outlook.)

At this point if there is no state to load we are done. The Security Manager has claimed that the object is safe to create and safe to run.  If there is persisted state then we have more work to do...

