# Asynchrony in C\# 5.0 part Four: It's not magic

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/4/2010 6:23:00 AM

-----

Today I want to talk about asynchrony that does not involve any multithreading whatsoever.

People keep on asking me "but how is it possible to have asynchrony without multithreading?" A strange question to ask because you probably already know the answer. Let me turn the question around: how is it possible to have multitasking without multiple CPUs? You can't do two things "at the same time" if there's only one thing doing the work\! But you already know the answer to that: multitasking on a single core simply means that the operating system stops one task, saves its continuation somewhere, switches to another task, runs it for a while, saves its continuation, and eventually switches back to continue the first task. *Concurrency* is an illusion in a single-core system; it is not the case that two things are really happening at the same time. How is it possible for one waiter to serve two tables "at the same time"? It isn't: the tables take turns being served. A skillful waiter makes each guest feel like their needs are met immediately by scheduling the tasks so that no one has to wait.

Asynchrony without multithreading is the same idea. You do a task for a while, and when it yields control, you do another task for a while *on that thread*. You hope that no one ever has to wait unacceptably long to be served.

Remember a while back I briefly sketched how early versions of Windows implemented multiple processes? Back in the day there was only one thread of control; each process ran for a while and then yielded control back to the operating system. The operating system would then loop around the various processes, giving each one a chance to run. If one of them decided to hog the processor, then the others became non-responsive. It was an entirely cooperative venture.

So let's talk about multi-threading for a bit. Remember a while back, in 2003, I talked a bit about the [apartment threading model](http://blogs.msdn.com/b/ericlippert/archive/2003/09/18/what-are-threading-models-and-what-threading-model-do-the-script-engines-use.aspx)? The idea here is that writing thread-safe code is expensive and difficult; if you don't have to take on that expense, then don't. If we can guarantee that only "the UI thread" will call a particular control then that control does not have to be safe for use on multiple threads. Most UI components are apartment threaded, and therefore the UI thread acts like Windows 3: everyone has to cooperate, otherwise the UI stops updating.

A surprising number of people have [magical beliefs](http://blogs.msdn.com/b/ericlippert/archive/2009/03/20/it-s-not-magic.aspx) about how exactly applications respond to user inputs in Windows. I assure you that it is not magic. The way that interactive user interfaces are built in Windows is quite straightforward. When something happens, say, a mouse click on a button, the operating system makes a note of it. At some point, a process asks the operating system "did anything interesting happen recently?" and the operating system says "why yes, someone clicked this thing."  The process then does whatever action is appropriate for that. What happens is up to the process; it can choose to ignore the click, handle it in its own special way, or tell the operating system "go ahead and do whatever the default is for that kind of event."  All this is typically driven by some of the simplest code you'll ever see:

while(GetMessage(\&msg, NULL, 0, 0) \> 0)  
{  
  TranslateMessage(\&msg);  
  DispatchMessage(\&msg);  
}

That's it. Somewhere in the heart of every process that has a UI thread is a loop that looks remarkably like this one. One call gets the next message. That message might be at too low a level for you; for example, it might say that a key with a particular keyboard code number was pressed. You might want that translated into "the numlock key was pressed". TranslateMessage does that. There might be some more specific procedure that deals with this message. DispatchMessage passes the message along to the appropriate procedure.

I want to emphasize that this is not magic. It's a while loop. [It runs like any other while loop in C that you've ever seen](http://blogs.msdn.com/b/oldnewthing/archive/2010/10/25/10080116.aspx). The loop repeatedly calls three methods, each of which reads or writes a buffer and takes some action before returning. If one of those methods takes a long time to return (typically DispatchMessage is the long-running one of course since it is the one actually doing the work associated with the message) then guess what? The UI doesn't fetch, translate or dispatch notifications from the operating system until such a time as it does return. (Or, unless some other method on the call chain is pumping the message queue, as Raymond points out in the linked article. We'll return to this point below.)

Let's take an even simpler version of our document archiving code from last time:

void FrobAll()  
{  
    for(int i = 0; i \< 100; ++i)  
        Frob(i);  
}

Suppose you're running this code as the result of a button click, and a "someone is trying to resize the window" message arrives in the operating system during the first call to Frob. What happens? Nothing, that's what. The message stays in the queue until everything returns control back to that message loop. The message loop isn't running; how could it be? It's just a while loop, and the thread that contains that code is busy Frobbing. The window does not resize until all 100 Frobs are done.

Now suppose you have

async void FrobAll()  
{  
    for(int i = 0; i \< 100; ++i)  
    {  
        await FrobAsync(i); // somehow get a started task for doing a Frob(i) operation on this thread  
    }  
}

What happens now?

Someone clicks a button. The message for the click is queued up. The message loop dispatches the message and ultimately calls FrobAll.

FrobAll creates a new task with an action.

The task code sends a message to its own thread saying "hey, when you have a minute, call me". It then returns control to FrobAll.

FrobAll creates an awaiter for the task and signs up a continuation for the task.

Control then returns back to the message loop. The message loop sees that there is a message waiting for it: please call me back. So the message loop dispatches the message, and the task starts up the action. It does the first call to Frob.

Now, suppose another message, say, a resize event, occurs at this point. What happens? Nothing. The message loop isn't running. We're busy Frobbing. The message goes in the queue, unprocessed.

The first Frob completes and control returns to the task. It marks itself as completed and sends another message to the message queue: "when you have a minute, please call my continuation". (\*)

The task call is done. Control returns to the message loop. It sees that there is a pending window resize message. That is then dispatched.

You see how async makes the UI more responsive without having any threads? Now you only have to wait for **one** Frob to finish, not for **all** of them to finish, before the UI responds. 

That might still not be good enough, of course. It might be the case that every Frob takes too long. To solve that problem, you could make each call to Frob itself spawn short asynchronous tasks, so that there would be more opportunities for the message loop to run. Or, you really could start the task up on a new thread. (The tricky bit then becomes posting the message to run the continuation of the task to the right message loop on the right thread; that's an advanced topic that I won't cover today.)

Anyway, the message loop dispatches the resize event and then checks its queue again, and sees that it has been asked to call the continuation of the first task. It does so; control branches into the middle of FrobAll and we pick up going around the loop again. The second time through, again we create a new task... and the cycle continues.

The thing I want to emphasize here is that we stayed on one thread the whole time. All we're doing here is breaking up the work into little pieces and sticking the work onto a queue; each piece of work sticks the *next* piece of work onto the queue. We rely on the fact that there's a message loop somewhere taking work off that queue and performing it.

**UPDATE**: A number of people have asked me "*so does this mean that the Task Asynchrony Pattern only works on UI threads that have message loops?*" No. The Task Parallel Library was explicitly designed to solve problems involving concurrency; task asynchrony extends that work. There are mechanisms that allow asynchrony to work in multithreaded environments without message loops that drive user interfaces, like ASP.NET. The intention of this article was to describe *how* asynchrony works on a UI thread without multithreading, not to say that asynchrony *only* works on a UI thread without multithreading. I'll talk at a later date about server scenarios where other kinds of "orchestration" code works out which tasks run when.

**Extra bonus topic**: Old hands at VB know that in order to get UI responsiveness you can use this trick:

Sub FrobAll()  
  For i = 0 to 99  
    Call Frob(i)  
    DoEvents  
  Next  
End Sub

Does that do the same thing as the C\# 5 async program above? **Did VB6 actually support continuation passing style?**

No; this is a much simpler trick. DoEvents does not transfer control back to the original message loop with some sort of "resume here" message like the task awaiting does. Rather, it starts up a *second* message loop (which, remember, is just a perfectly normal while loop), clears out the backlog of pending messages, and then returns control back to FrobAll. Do you see why this is potentially dangerous?

What if we are in FrobAll as a result of a button click? And what if while frobbing, the user pressed the button again? DoEvents runs another message loop, which clears out the message queue, and now we are running FrobAll within FrobAll; it has become reentrant. And of course, it can happen again, and now we're running a third instance of FrobAll...

Of course, the same is true of task based asynchrony\! If you start asynchronously frobbing due to a button click, and there is a second button click while more frobbing work is pending, then you get a second set of tasks created. To prevent this it is probably a good idea to make FrobAll return a Task, and then do something like:

async void Button\_OnClick(whatever)  
{  
    button.Disable();  
    await FrobAll();  
    button.Enable();  
}

so that the button cannot be clicked again while the asynchronous work is still pending.

**Next time**: Asynchrony is awesome but not a panacea: a real life story about how things can go terribly wrong.

\------------

(\*) Or, it invokes the continuation right then and there. Whether the continuation is invoked aggressively or is itself simply posted back as more work for the thread to do is user-configurable, but that is an advanced topic that I might not get to.

