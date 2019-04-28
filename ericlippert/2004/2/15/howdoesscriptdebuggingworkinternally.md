# How Does Script Debugging Work Internally?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/15/2004 11:23:00 AM

-----

Every now and then someone will ask me how the behind-the-scenes magic works that allows you to debug scripts. 

Darned if I know.  While the rest of the script team was busy implementing the debugging interfaces, I was working on the engines.  I understand only a little of the specifics.  But I can give you an overview. 

The debugging architecture has five parts: 

  - the **host** (IE, ASP, WSH, other third party hosts)

  - the **engine** (VBScript, JScript)

  - the **process debug manager** (PDM.DLL)

  - the **machine debug manager** (MDM.EXE)

  - the **debugger** (Visual Studio, Visual Interdev, the Script Debugger) 

There are two kinds of hosts: "smart" and "dumb".  Smart hosts know how the debugger architecture works, dumb hosts do not.  The script engine can detect whether it is running in a smart or dumb host.  (It asks the host "are you smart?" and if the answer is "Huh?" then obviously its not.  :-) )  If it's running in a dumb host, then the script engine takes care of the debugging plumbing.  If it is running in a smart host, then the engine and the host each have a part in the plumbing.  

The reasoning behind this is that we wanted to enable script developers to debug their scripts even if they were running in third party "dumb" hosts, but we also want smart hosts to be able to do things like provide context.  When you debug a script in a web page, the script engine knows nothing about the HTML in which it is embedded -- it needs the cooperation of IE to tell the debugger what HTML file to display.  Similarly, ASP and WSH are smart; they provide information about the context in which the script is running. 

The script engine and the host do not talk to the debugger directly.  Instead, they talk to an in-process component, aptly named the Process Debug Manager.  The PDM knows how to talk to the Machine Debug Manager, which runs as its own process.  The MDM acts as a dating service to hook up willing debuggers with willing debugees. 

To sum up the story so far: 

  - The (smart) host knows how to talk to the engine and the PDM. 

  - The engine knows how to talk to the host and the PDM. 

  - The PDM knows how to talk to the MDM.

  - The MDM knows how to talk to the PDM and the debugger.

  - The debugger knows how to talk to the MDM. 

As you can see, the debugging architecture was designed for **extensibility** and **abstraction**.  A debuggee need know nothing about the debuggers, and vice-versa.  The debuggee need know nothing about what debuggers are installed on the machine, etc.  It just talks to the process debug manager, the PDM contacts the MDM, and everyone gets hooked up. 

And now you know everything that I know about how script debugging works.  (OK, maybe I know a *little bit* more than that.  Someday I might do a blog entry on how to turn a dumb host into a smart host.)

Want to know more?  See the MSDN documentation [here](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/script56/html/conactivescriptdebuggingoverview.asp).

