# How To Obtain The Name Of The Client From The ASP Server

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/9/2005 3:18:00 PM

-----

Here's a question about client side vs. server side scripting that I got recently:

*I want to get the machine name of the client the request is being made from. With ASP I can get the IP address using this code: ipaddr = Request.ServerVariables("REMOTE\_ADDR") But I don’t know how to get the name of the machine. Is there something I could do from the client side?*

No, the web browser client cannot determine the name of the machine for two reasons.

First, **if it could then the client could be instructed to send the name of the machine to an evil server**. Evil hackers would love to have an *internet* web page that harvested *intranet* machine names that they could then attack. Knowing the name of a machine is particularly useful for social engineering attacks -- if someone phoned me up claiming to be from our IT department, I'd be a lot more inclined to believe them if they knew the names of all my machines.

Second, look at it from the other way.  Suppose the client magically figures out its name and sends it to the server.  *Why should the server trust the client?*  What stops an evil client from sending a bogus name to the server? Even if the client could send the name, the server can't make any decisions based on that name, so it's kind of useless.

Clients and servers should not trust each other.  In the absence of authentication evidence, clients must assume that all servers are run by evil hackers and servers must assume that all clients are run by evil hackers.  Once you accept that fundamental design principle then it becomes much easier to reason about client-server interactions. Think like an evil person\!

Another developer who saw this question suggested running this code on the server:

 

name = Request.ServerVariables("REMOTE\_HOST")

That's a good start but not the whole story. By default this doesn't actually give you the remote host -- it just gives you the IP address again. If you want this to actually give you the name of the remote machine then there's some additional work you have to do. Since we have the IP address then we can do a reverse DNS lookup to see if there is a friendly name associated with that address. Now the server is trusting not an arbitrarty client but rather a specific reverse DNS server.

Read this Knowledge Base article on how to configure your server to automatically do Reverse DNS lookups when the code above is called.

<http://support.microsoft.com/default.aspx?scid=kb;en-us;Q245574>

Note that this will make your server performance worse, and of course is not guaranteed to work if the client machine is disguising its identity via a firewall, etc.

