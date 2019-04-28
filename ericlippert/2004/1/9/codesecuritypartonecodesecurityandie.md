# Code Security Part One: Code Security and IE

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/9/2004 10:37:00 AM

-----

I want to start this year by rambling on a bit about security and script, how various code-based and role-based security systems work, and so on.  (I'll probably get back to rambling on about the JScript .NET type system at a later date.)  Those of you who managed to snap up a copy of my book back when Wrox was solvent will probably get a sense of déjà vu -- but that's a pretty small number of people.   

Let me start by talking a bit about **code security in IE.** 

The Windows XP security system is designed to restrict the rights of users by **authenticating** those users, determining what they are **authorized** to do, and ensuring that **all programs that run on their behalf are given the same rights as the**** user**. Suppose that you log in and start up Internet Explorer, then browse to a random web page on the Internet that contains some script: 

\<script language="vbscript"\>  
Set FSO = CreateObject("Scripting.FileSystemObject")  
FSO.DeleteFile "C:Blah.txt", True  
\</script\> 

It should be obvious that this script would attempt to delete a (possibly vital) file from your file system. The script runs in Internet Explorer, which (like any other process you start) has your "security token". If **you** have the right to delete that file then **IE** does too, as far as Windows is concerned. That might not be what you wanted to happen; perhaps you just wanted to surf the web unmolested by people trying to interfere with your file system. 

Clearly, **it is undesirable to offer all your privileges to anyone who happens to put up a web page**. But on the other hand, to prevent **all** web page script code from running would make the web a dull place. After all, most pages with script are harmless and use the script for benign purposes. 

This leads us to our first important security principle: **the usability of a system is proportional to its security**.  Highly secure systems are, by definition, restrictive systems.  Cars that need keys are less usable than cars with no locks at all.  Finding the right balance between security and usability is one of the most difficult tasks for security-minded developers. 

The way we ensure that systems are both secure and usable is to figure out what the software is allowed to do, and allow only that. This is known as the principle of least privilege:  **secure software grants only those rights necessary to do what the user wants. **

IE solves this problem by implementing its own security system **independent** **of the underlying Windows security model**. 

In IE's code-based security model the "server address" portion of the URL of every page is noted. **All the scripts and objects created by a page are "tagged" with the server address of the page.** IE assigns each page into a certain **zone** based on the origin of the page. IE has five zones: **Internet**, **Intranet**, **Trusted**, **Restricted**, and a special implicit zone called **Local Machine**, which is used for code run from the users own machine. You can put sites for which you wish to allow less stringent security checks into the Trusted zone; and similarly, put web sites that you suspect may be hostile into the Restricted zone. 

When IE runs a script, it determines what restrictions should be placed upon the script by checking the zone of the page. For instance, a page loaded from a Trusted site might be granted the right to create potentially dangerous objects (such as the File System Object). That same page might produce a warning dialog when run from the Internet Sites zone and an error when run from the Restricted zone. (The actual behavior is configurable in the Security tab of the IE options dialog.) 

You might wonder why IE does not simply use the Windows security system to solve the problem of web sites doing unwanted things. For instance, IE could modify its own security token, removing privileges that hostile pages need to do dangerous things. Perhaps it could create a security token with no rights to read the disk or do other harmful things, and then run the browser in a thread with this token rather than the user's token. That would be a pretty good idea, and in fact, that's the idea behind Software Restriction Policies, a security feature introduced in Windows XP (which I may delve into in more detail later.) 

Unfortunately, that solution would not completely solve all the security problems associated with web browsing. Consider this scenari 

Imagine a page with a navigation bar frame on the left and a browsing frame beside it. If the user navigates the browsing frame to a different web site then **the script running from the navigation bar frame must not be able to access any information about the browsing frame**. If it could then hostile web page authors could create a page with an **invisible** "navigation bar" frame in order to read information about all the web sites you visit and **post that private information** back to the invisible frame's host. That would lead to potentially huge vulnerabilities; if you type a password into the browser then you assume that only the site to which you posted the password can actually receive it. 

The problem essentially is that **pages from disparate sites must be able to run scripts in the same browser without accessing any information about each other**. This problem cannot easily be solved with Windows user-based security. The Windows security system was designed to restrict the rights of users, not pages in framesets. IE's security system needs to be more granular and flexible. 

Thus, the Internet Explorer implementers had to develop their own security system, which tracks the origins of all pages in the frameset, and prevents pages from one site from accessing any information about pages from a different site. Once a system for tracking the origin of every page is in place then it is relatively easy to categorize those pages into zones and restrict their rights accordingly. 

One can think about the behavior of the IE security system quite independently of the Windows security system. However, even though IE's security system is conceptually independent of the Windows security system, both are in effect while IE is running\! Suppose, for example, you browse to a page in the Intranet zone. Suppose further that the IE security system grants the page the right to load information from a particular file stored in a shared directory somewhere on your intranet. Since IE is running with your security token, the attempt to read the file will still fail if your account does not have permission to do so. The Windows security system is always on, even when other security systems such as the IE or .NET CLR security systems are on as well. 

IE's security mechanism is complex and highly configurable; we have barely scratched the surface. The details of any one aspect alone, such as how exactly objects are downloaded and created would take us far afield. The point I want to get across is this: **Windows restricts the rights of users by authenticating their identities and granting them rights accordingly. IE's security system by contrast restricts the rights of individual chunks of code (on web pages) based on evidence describing the origins of the pages. It also restricts the ways in which these chunks of code may interact. **

The .NET CLR security system also needs to solve the same basic three problems: to restrict the rights of programs based on **who** is running them, to restrict the rights of chunks of code based on **where** they came from, and to restrict the ways in which code from disparate locations **interacts**.  More on all of that coming soon\!

