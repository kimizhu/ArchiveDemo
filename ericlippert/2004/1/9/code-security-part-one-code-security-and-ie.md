<div id="page">

# Code Security Part One: Code Security and IE

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/9/2004 10:37:00 AM

-----

<div id="content">

<span>I want to start this year by rambling on a bit about security and script, how various code-based and role-based security systems work, and so on.<span>  </span>(I'll probably get back to rambling on about the JScript .NET type system at a later date.)<span>  </span>Those of you who managed to snap up a copy of my book back when Wrox was solvent will probably get a sense of déjà vu -- but that's a pretty small number of people.<span>  </span> </span>

<span></span>

<span>Let me start by talking a bit about **<span>code security in IE.</span>** </span>

<span></span>

<span>The Windows XP security system is designed to restrict the rights of users by **<span>authenticating</span>** those users, determining what they are **<span>authorized</span>** to do, and ensuring that **<span>all programs that run on their behalf are given the same rights as the</span>**</span>**<span><span> </span>user</span>**<span>. Suppose that you log in and start up Internet Explorer, then browse to a random web page on the Internet that contains some script: </span>

<span></span>

<span>\<script language="vbscript"\>  
</span><span>Set FSO = CreateObject("Scripting.FileSystemObject")  
</span><span>FSO.DeleteFile "C:Blah.txt", True  
</span><span>\</script\> </span>

<span></span>

<span>It should be obvious that this script would attempt to delete a (possibly vital) file from your file system. The script runs in Internet Explorer, which (like any other process you start) has your "security token". If **<span>you</span>** have the right to delete that file then **<span>IE</span>** does too, as far as Windows is concerned. That might not be what you wanted to happen; perhaps you just wanted to surf the web unmolested by people trying to interfere with your file system. </span>

<span></span>

<span>Clearly, **<span>it is undesirable to offer all your privileges to anyone who happens to put up a web page</span>**. But on the other hand, to prevent **<span>all</span>** web page script code from running would make the web a dull place. After all, most pages with script are harmless and use the script for benign purposes. </span>

<span></span>

<span>This leads us to our first important security principle: **<span>the usability of a system is proportional to its security</span>**.<span>  </span>Highly secure systems are, by definition, restrictive systems.<span>  Cars that need keys are less usable than cars with no locks at all.  </span>Finding the right balance between security and usability is one of the most difficult tasks for security-minded developers. </span>

<span></span>

<span>The way we ensure that systems are both secure and usable is to figure out what the software is allowed to do, and allow only that.<span> </span>This is known as the principle of least privilege:<span>  </span>**<span>secure software grants only those rights necessary to do what the user wants. </span>**</span>

<span></span>

<span>IE solves this problem by implementing its own security system **<span>independent</span>** **<span>of the underlying Windows security model</span>**. </span>

<span></span>

<span>In IE's code-based security model the "server address" portion of the URL of every page is noted. **<span>All the scripts and objects created by a page are "tagged" with the server address of the page.</span>** IE assigns each page into </span><span>a certain **<span>zone</span>** based on the origin of the page. IE has five zones: **<span>Internet</span>**, **<span>Intranet</span>**, **<span>Trusted</span>**, **<span>Restricted</span>**, and a special implicit zone called **<span>Local Machine</span>**, which is used for code run from the users own machine. You can put sites for which you wish to allow less</span><span><span> </span>stringent security checks into the Trusted zone; and similarly, put web sites that you suspect may be hostile into the Restricted zone. </span>

<span></span>

<span>When IE runs a script, it determines what restrictions should be placed upon the script by checking the zone of the page. For instance, a page loaded from a Trusted site might be granted the right to create potentially dangerous objects (such as the File System Object). That same page might produce a warning dialog when run from the Internet Sites zone and an error when run from the Restricted zone. (The actual behavior is configurable in the Security tab of the IE options dialog.) </span>

<span></span>

<span>You might wonder why IE does not simply use the Windows security system to solve the problem of web sites doing unwanted things. For instance, IE could modify its own security token, removing privileges that hostile pages need to do dangerous things. Perhaps it could create a security token with no rights to read the disk or do other harmful things, and then run the browser in a thread with this token rather than the user's token. That would be a pretty good idea, and in fact, that's the idea behind Software Restriction Policies, a security feature introduced in Windows XP (which I may delve into in more detail later.) </span>

<span></span>

<span>Unfortunately, that solution would not </span><span>completely solve all the security problems associated with web browsing. Consider this scenari </span>

<span></span>

<span>Imagine a page with a navigation bar frame on the left and a browsing frame beside it. If the user navigates the browsing frame to a </span><span>different web site then **<span>the script running from the navigation bar frame must not be able to access any information about the browsing frame</span>**. If it could then hostile web page authors could create a page with an **<span>invisible</span>** "navigation bar" frame in order to read information about all the web sites you visit and **<span>post that private information</span>** back to the invisible frame's host. That would lead to potentially huge vulnerabilities; if you type a password into the browser then you assume that only the site to which you posted the password can actually receive it. </span>

<span></span>

<span>The problem essentially is that **<span>pages from disparate sites must be able to run scripts in the same browser without accessing any information about each other</span>**. This problem cannot easily be solved with Windows user-based security. The Windows security system was designed to restrict the rights of users, not pages</span><span><span> </span>in framesets. IE's security system needs to be more granular and flexible. </span>

<span></span>

<span>Thus, the Internet Explorer implementers had to develop their own security system, which tracks the origins of all pages in the frameset, and prevents pages from one site from accessing any information about pages from a different site. Once a system for tracking the origin of every page is in place then it is relatively easy to categorize those pages into zones and restrict their rights accordingly. </span>

<span></span>

<span>One can think about the behavior of the IE security system quite independently of the Windows security system. However, even though IE's security system is conceptually independent of the Windows security system, both are in effect while IE is running\! Suppose, for example, you browse to a page in the Intranet zone. Suppose further that the IE security system grants the page the right to load information from a particular file stored in a shared directory somewhere on your intranet. Since IE is running with your security token, the attempt to read the file will still fail if your account does not have permission to do so. The Windows security system is always on, even when other security systems such as the IE or .NET CLR security systems are on as well. </span>

<span></span>

<span>IE's security mechanism is complex and highly configurable; we have barely scratched the surface. The details of any one aspect alone, such as how exactly objects are downloaded and created would take us far afield. The point I want to get across is this: **<span>Windows restricts the rights of users by authenticating their identities and granting them rights accordingly. IE's security system by contrast restricts the rights of individual chunks of code (on web pages) based on evidence describing the origins of the pages. It also restricts the ways in which these chunks of code may interact. </span>**</span>

<span></span>

<span>The .NET CLR security system also needs to solve the same basic three problems: to restrict the rights of programs based on **<span>who</span>** is running them, to restrict the rights of chunks of code based on **<span>where</span>** they came from, and to restrict the ways in which code from disparate locations **<span>interacts</span>**.<span>  </span>More on all of that coming soon\! </span>

</div>

</div>

