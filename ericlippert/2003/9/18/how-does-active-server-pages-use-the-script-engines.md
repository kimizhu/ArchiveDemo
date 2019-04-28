<div id="page">

# How does Active Server Pages use the script engines?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 5:32:00 PM

-----

<div id="content">

<span>It's always struck me as a little bit odd that Active Server Pages, a web server, encourages developers to use VBScript and JScript to write server-side scripts.<span>  </span>I mean, the whole point of a web server is that it produces complex strings (web pages are just strings of HTML after all) as blindingly fast as possible, on demand.<span>  </span>"Blindingly fast" and "script language" do not really go together, compared to, say, C. </span>

<span></span>

<span>My opinions -- and you'd better believe I have plenty of strong ones\! -- on performance characteristics of the script engines will certainly come back in [future posts](http://blogs.msdn.com/ericlippert/category/2717.aspx?Show=All).<span>  </span>But for now I want to talk a little bit about what features we added to the script engines to make them work as well as they do for ASP. </span>

<span></span>

<span>One of the ways you make a server fast is by caching everything you possibly can cache.<span>  </span>For example, suppose you need to do a job on a separate thread.<span>  </span>You'd probably create a thread, do some work on the thread, and throw it away when you're done.<span>  </span>OK, now suppose you need to do a million jobs but never more than, say, three at a time. Creating and destroying those million threads is potentially going to be a not-insignificant amount of the total processor time spent.<span>  </span>You'd be better off **<span>creating three threads and re-using them as necessary</span>**. </span>

<span></span>

<span>This strategy is called “thread pooling”, and ASP certainly uses it.</span>

<span>(UPDATE: I've written a follow-up article on [Thread Happiness Disease](http://blogs.msdn.com/ericlippert/archive/2004/02/15/73366.aspx).)</span>

<span>ASP caches a lot more than that though.<span>  </span>It also pools script engines and caches compiled state.<span>  </span>Let me explain with an example. </span>

<span></span>

<span>Suppose a request comes in for this page, time.asp:</span>

<span> </span><span>\<html\></span><span>  
</span><span>\<% Response.write Now() %\></span><span>  
</span><span>\</html\></span><span> </span>

<span>Suppose further that this is the first-ever request for this page since the server was started.  </span>

<span></span>

<span>The thing above isn’t a legal script, so ASP parses out the non-script blocks and stores them in the Response object.<span>  </span>It then generates a real script that looks something like </span>

<span></span>

<span>Response.WriteBlock 0</span><span>  
</span><span>Response.Write Now()</span><span>  
</span><span>Response.WriteBlock 1</span><span> </span>

<span></span>

<span>ASP also has a pool of created-but-not-initialized script engines sitting around.<span>  </span>I said in my last post that **when the script engines are not initialized they can be called on any thread,** so it does not matter that these engines were not created on whatever thread happened to be pulled out of the thread pool.<span>  </span></span>

<span><span></span></span>

<span><span></span>ASP initializes the engine on the current thread and passes in the script code above.<span>  </span>The script engine compiles the script, runs it, and the ASP object model sends the resulting string off to the client. </span>

<span></span>

<span>The script engine then gets uninitialized, but does not throw away its compiled state.<span>  </span>The compiled state – the proprietary bytecode form of the language – is maintained in memory because the second time someone asks for that page, ASP do not want to have to go through all the time and expense of translating the page to script and compiling the script to bytecode. </span>

<span></span>

<span>The thread and the now-uninitialized engine go back to their respective pools. </span>

<span></span>

<span>Now suppose a second request comes in for time.asp.<span>  </span>This time ASP pulls a thread out of the thread pool, notices that it has an engine with the compiled state just sitting there, attaches the engine to the thread, and runs the script again.<span>  </span>No time is spent creating the thread, creating the script engine, translating the page to code or compiling the script to bytecode. </span>

<span></span>

<span>Suppose further that in the few milliseconds that ASP is running this script, a third request comes in for this page.<span>  </span>Then what happens?<span>  </span>There are two obvious possibilities: </span>

<span></span>

<span><span>1)<span>      </span></span></span><span>Wait for the current engine to finish, and start it right back up again when it is done.<span>   
</span></span><span><span>2)<span>      </span></span></span><span>Fetch a new engine from the engine pool and start over – translate the page to script, compile, run. </span>

<span></span>

<span>The trouble is, both of those are potentially slow.<span>  </span>The first in particular could be extremely slow. We needed a third option. </span>

<span></span>

<span>In my previous post I discussed [the sorts of trouble you can get into with multi-threading](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx).<span>  </span>Note that the problem I described happens because two threads tried to both read *<span>and</span>* write a particular value.<span>  </span>If the value was read-only, then obviously there would not have been a problem.<span>  </span>You can read a read-only value from as many threads as you want\! </span>

<span></span>

<span>Hey, once the script engine compiles a script into bytecode, that bytecode is read-only.<span>  </span>It’s never going to change again.<span>  </span>This means that two script engines on two different threads can share bytecode\! </span>

<span></span>

<span>I lied earlier when I said that </span><span>InterruptScriptThread</span><span> was the only exception to our weird threading model.<span>  </span>We have another exception – at any time, you can call </span><span>Clone</span><span> from any thread.<span>  </span>This takes an existing script engine and gives you a new script engine with exactly the same compiled state, but able to run on a different thread. Each engine maintains its own runtime state – variable values, etc – but the compiled state is shared. </span>

<span></span>

<span>So in this scenario, ASP **<span>clones the running engine onto another thread and then runs the new engine concurrently</span>**.<span>  </span>This is *somewhat* expensive – it has to create a new engine, after all – but is *much cheaper* than compiling up all that state again. </span>

<span></span>

<span>Of course, ASP is much more complicated than that quick sketch.<span>  </span>What I’m getting at here is that there actually is some rhyme and reason to our bizarre-seeming choices of threading model.<span>  </span></span>

<span><span></span></span>

<span><span></span>The script engine design was driven by the need to solve a very specific set of very disparate problems.  Without knowing what the problems were, the resulting design looks kind of random.  But once you know the history, well, I guess it looks a little *less* random.</span>

<span>UPDATEL: I've written a follow-up article on [how the ASP compilation model is affected by having multiple languages per page](http://blogs.msdn.com/ericlippert/archive/2004/02/19/76438.aspx).</span>

</div>

</div>

