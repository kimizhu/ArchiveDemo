# Continuation Passing Style Revisited Part Five: CPS and Asynchrony

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/27/2010 6:45:00 AM

-----

Today is when things are going to get really long and confusing. But we'll make it through somehow. Consider the following task: you’ve got a list of URLs. You want to fetch the document associated with each URL. (Let’s suppose for the sake of argument that this always succeeds.) You then want to make a copy of the document on your network attached tape drive storage, because you’re *old school*. Totally straightforward. Two lines of code. Hardly worth making a method out of:  

void ArchiveDocuments(List\<Url\> urls)  
{  
  for(int i = 0; i \< urls.Count; ++i)  
    Archive(Fetch(urls\[i\]));  
}

Super. Trouble is, if the list of URLs is long and the network connection is slow then the control flow is going to go like this:

  - Fetch the first document.
  - Wait, wait, wait, wait, wait...
  - OK, got the first document. Archive it.
  - Wait, wait, wait, wait...
  - OK, it’s archived. Fetch the second document.
  - Wait, wait, wait, wait…
  - OK, got the second document. Archive it.
  - Wait, wait, wait, wait...

You spend most of your time waiting with the CPU doing nothing at all. You might reasonably suppose that **while you’re waiting for the first document to archive to tape, you could also be waiting for the second document to arrive from the network**. But how on earth are you going to do that? Well, we know that **all possible control flows can be built out of continuations**. We know that a continuation is essentially a delegate that represents “what comes next”. Let’s suppose that a miracle happens and some kind person gives us a method FetchAsync that implements Fetch using CPS:  

void FetchAsync(Url url, Action\<Document\> continuation)

FetchAsync somehow – who knows how, that’s its business – manage to asynchronously do its task. Maybe it grabs a thread from a thread pool, or spins up an I/O completion thread, or it does some Windows magic to get completion messages posted to it on this thread’s message loop. I don’t know and right now I don’t care. All I care about is that its contract is that (1) somehow it manages to do this task asynchronously, and (2) when the asynchronous task is complete it calls the continuation it has been given.

Notice that the document fetcher’s continuation takes a document; remember, calling a continuation is logically equivalent to returning a value from a subroutine call, which is what we were doing in the synchronous version. Super fantastic. Let’s start rewriting this thing in something similar to CPS and see what happens. Immediately we run into a difficulty when attempting to write the continuation in C\#: how on Earth are we going to keep going around the loop where we left off, with the value of "i" preserved correctly? Apparently we need to solve two problems:

  - What were the values of all the local variables when last we were here?
  - Where does control flow pick up, and how?

The first problem can be solved by **making a delegate which closes over the local variables**; we already know that doing so captures local variables and hoists them into a closure. The second problem is the same problem faced by iterator blocks. After all, they have to solve the same problem: resume control after a yield return. We saw yesterday Raymond's articles on how the C\# compiler does that: it generates a state machine and a bunch of "gotos" to branch to the right places. Clearly we do not want to perform a full-on wholesale translation of this method into Continuation Passing Style. The cognitive burden and performance cost is simply too high. Rather, we can use the same trick we used for iterator blocks. We'll solve both problems by combining the solutions: let's make a lambda so that we get the closure, and rewrite its interior as a state machine that implements the continuation without rewriting everything in CPS:  

enum State { Start, AfterFetch }  
  
void ArchiveDocuments(List\<Url\> urls)  
{  
  State state = State.Start;  
  int i;  
  Document document;  
  Action archiveDocuments = () =\>  
  {  
    switch(state)  
    {  
      case State.Start:      goto Start;  
      case State.AfterFetch: goto AfterFetch;  
    }  
    Start: ;  
    for(i = 0; i \< urls.Count; ++i)  
    {  
      state = State.AfterFetch;  
      FetchAsync(urls\[i\],  
        resultingDocument=\>  
        {  
          document = resultingDocument;  
          archiveDocuments();  
        });  
      return;  
      AfterFetch: ;  
      Archive(document);  
    }  
  };  
  archiveDocuments();  
}

The attentive reader will note that we have multiple problems here. First, we have a [definite assignment problem because we have a recursive lambda](http://blogs.msdn.com/b/ericlippert/archive/2006/08/18/706398.aspx). Let’s ignore that for now. Second, we have a branch from outside to inside a block, which is illegal in C\#. Let's ignore that too. Let's review the story so far. Someone calls ArchiveDocuments with a bunch of URLs. That creates an action that does the real work of the method, and invokes it. The “real” method then dispatches control to the Start label. FetchAsync starts fetching the first document asynchronously. Since its contract is that it returns immediately and does its work asynchronously, there’s now nothing more we can do here. *We need to await the coming of the document*. We return, and know that we’ll be called again when the document is fetched. When the document is fetched, the continuation is invoked. It mutates the closure to have the latest copy of the document. We’ve already mutated the state so that we know where to pick up again. The continuation then invokes the method. The method branches to the AfterFetch label. (Again, ignoring that technically this is illegal code.) We jump into the middle of the loop and pick up where we left off. We *synchronously* archive the document. When it is archived, we branch back up to the top of the loop, start *asynchronously* fetching the next document, and the whole thing repeats. This is a step in the right direction, but obviously we haven’t achieved our goal yet. We need to asynchronously archive the document as well. Couldn’t we just do the same thing we did before? That is, invent a magic  

void ArchiveAsync(Document document, Action continuation)

method, and then transform the program above to use it? (The original Archive(document) is a void-returning method so in the asynchronous version its continuation takes no argument.) Let’s try to do so naively and see what happens.  

Action archiveDocuments = () =\>  
{  
  switch(state)  
  {  
    case State.Start:        goto Start;  
    case State.AfterFetch:   goto AfterFetch;  
    case State.AfterArchive: goto AfterArchive;  
  }  
  Start: ;  
  for(i = 0; i \< urls.Count; ++i)  
  {  
    state = State.AfterFetch;  
    FetchAsync(urls\[i\], resultingDocument=\>  
    {  
      document = resultingDocument;  
      archiveDocuments();  
    });  
    return;  
    AfterFetch: ;  
    state = State.AfterArchive;  
    ArchiveAsync(document, archiveDocuments);  
    return;  
    AfterArchive: ;  
  }  
};

OK, now what happens? We fetch asynchronously and return immediately. When the fetch completes, its completion branches to the archive step. It starts archiving asynchronously and immediately returns. When the archive completes, its completion branches to the end of the loop, and everything starts over. Drat. We forgot to actually implement the feature we need: that the wait time for the second fetch should overlap the wait time for the first document to be archived. We’re returning immediately after starting the archiving task. Something is missing. What we want to do is start the archiver, then start the next fetch, then wait for them both to complete *before* starting to archive the next document. The problem here is that we’re still treating ArchiveAsync as though it were synchronous for the purposes of determining what happens next; all we’ve done so far is greatly complicate the *actual* control flow of this method without actually adding any new *logical* control flow. What we need is to be able to start the asynchronous archive, **not return**, and then do other stuff – like, start the next fetch. If we're not returning immediately then that means that the asynchronous start should not take a continuation, because "the thing that happens next" is about to happen\! We're *not returning*. So what takes the continuation? Clearly our methods FetchAsync and ArchiveAsync are insufficient to meet our needs. Let’s suppose that instead of being void, rather FetchAsync returns, uh, AsyncThingy\<Document\>, a type which I just magically invented that represents “*I am tracking the state of an asynchronous operation that is going to someday produce a document*”. It’s the *thingy* that takes the continuation. Similarly, ArchiveAsync can return AsyncThingy, with no type argument, because it is void returning. Let’s rewrite our method using that:  

AsyncThingy\<Document\> fetchThingy = null;  
AsyncThingy archiveThingy = null;  
  
Action archiveDocuments = () =\>  
{  
  switch(state) { blah blah blah }  
  Start: ;  
  for(i = 0; i \< urls.Count; ++i)  
  {  
    fetchThingy = FetchAsync(urls\[i\]);  
    state = State.AfterFetch;  
    fetchThingy.SetContinuation(archiveDocuments);  
    return;  
    AfterFetch: ;  
    document = fetchThingy.GetResult();  
    archiveThingy = ArchiveAsync(document);  
    state = State.AfterArchive;  
    archiveThingy.SetContinuation(archiveDocuments);  
    return;  
    AfterArchive: ;  
  }  
};

Better, but not there yet. Now we have basically the same control flow as before. We’ve eliminated one nested lambda though, because now we can get the document out of the fetch thingy after the continuation completes. We don’t have to have some mechanism whereby the continuation mutates the state, which is nice. But what about the feature we’re after, namely, *sharing the wait tim*e for the previous document archiving with the current document fetch? What we need to do is *not set the continuation of the archive task until later:*  

  Start: ;  
  for(i = 0; i \< urls.Count; ++i)  
  {  
    fetchThingy = FetchAsync(urls\[i\]);  
    state = State.AfterFetch;  
    fetchThingy.SetContinuation(archiveDocuments);  
    return;  
    AfterFetch: ;  
    document = fetchThingy.GetResult();  
    if (archiveThingy \!= null)  
    {  
      state = State.AfterArchive;  
      archiveThingy.SetContinuation(archiveDocuments);  
      return;  
      AfterArchive: ;  
    }  
    archiveThingy = ArchiveAsync(document);  
  }  

Is this what we want? Let’s go through it. First time through we start asynchronously fetching a document. We return immediately. When the document is fetched, the continuation is invoked and we branch to AfterFetch. There is no archive thingy yet, so we skip the consequence and move on. We get the fetched document and start archiving it asynchronously. We do not return at this point. We go back up to the top of the loop and start asynchronously fetching the next document. *We are now waiting for fetching and archiving at the same time*. The method returns immediately after setting the continuation of the fetch.

When the fetch of the second document is complete, its continuation is invoked. That continuation branches to AfterFetch. This time there is a *previously-started archive task thingy*, so we set its continuation and immediately return. When the archive task completes then we know that *both the current fetch task and the previous archive task have completed*, so we go on to start archiving the current document. We start that asynchronously and then loop back around and start asynchronously fetching the next document, and things repeat as needed. Whew. Could we do better? Something seems not quite right here. What if the archive of the first document *completed* while the second document was being *fetched*? In that case we would not *need* to go through all the rigamarole of assigning a continuation and then immediately returning, only to have the continuation immediately invoked. Similarly, suppose FetchAsync maintains its own local cache on the current machine, and some other process has already fetched that document recently; perhaps the fetcher can sometimes complete *immediately* without doing any expensive asynchronous operation. To handle these scenarios, let’s suppose that SetContinuation returns a bool indicating whether the task is already completed yet or not. True if the task still has asynchronous work to do, false otherwise. (That is, true if the thingy is actually going to invoke the continuation at a later date, false if it is not.) Let's write the whole thing in its final form: 

void ArchiveDocuments(List\<Url\> urls)  
{  
  State state = State.Start;  
  int i;  
  Document document;  
  AsyncThingy\<Document\> fetchThingy = null;  
  AsyncThingy archiveThing = null;  
  Action archiveDocuments = () =\>  
  {  
    switch(state)  
    {  
      case State.Start:        goto Start;  
      case State.AfterFetch:   goto AfterFetch;  
      case State.AfterArchive: goto AfterArchive;  
    }  
    Start: ;  
    for(i = 0; i \< urls.Count; ++i)  
    {  
      fetchThingy = FetchAsync(urls\[i\]);  
      state = State.AfterFetch;  
      if (fetchThingy.SetContinuation(archiveDocuments))  
      return;  
      AfterFetch: ;  
      document = fetchThingy.GetResult();  
      if (archiveThingy \!= null)  
      {  
        state = State.AfterArchive;  
        if (archiveThingy.SetContinuation(archiveDocuments))  
          return;  
        AfterArchive: ;  
      }  
      archiveThingy = ArchiveAsync(document);  
    }  
  };  
  archiveDocuments();  
}

And we’re done. Take a good look at that code in comparison with what we started with:  

void ArchiveDocuments(List\<Url\> urls)  
{  
  for(int i = 0; i \< urls.Count; ++i)  
    Archive(Fetch(urls\[i\]));  
}

Holy goodness, what a godawful mess we’ve made. We've expanded two lines of perfectly clear code into two dozen lines of the most godawful spaghetti you've ever seen. And of course it still doesn’t even *compile* because the labels aren’t in scope and we have a definite assignment error. We;d still need to further rewrite the code to fix those problems. Remember what I was saying about the pros and cons of CPS?

  - PRO: Arbitrarily complex and interesting control flows can be built out of simple parts – check.
  - CON: The reification of control flow via continuations is hard to read and hard to reason about – check.
  - CON: The code that represents the mechanisms of control flow completely overwhelms the meaning of the code – check.
  - CON: The transformation of ordinary code control flow into CPS is the kind of thing that compilers are good at, and almost no one else is – check.

**This is not some intellectual exercise**. **Real people end up writing code morally equivalent to the above all the time when they deal with asynchrony.** And, ironically, even as processors have gotten faster and cheaper, we spend more and more of our time *waiting for stuff that isn’t processor-bound*. In many programs, much of the time spent is with the processor pegged to zero waiting for network packets to make the trip from England to Japan and back again, or for disks to spin, or whatever. And I haven't even talked about what happens if you want to compose asynchronous operations further. Suppose you want to make ArchiveDocuments an asynchronous operation that takes a delegate. Now all the code that calls it has to be written in CPS as well. The taint just spreads. Getting asynchronous logic right is **important**, it’s only going to be **more important in the future**, and the tools we have given you make you “*stand on your head and turn yourself inside out*” as Professor Duggan wisely said. Continuation passing style is powerful, yes, but there's got to be a better way of making good use of that power than the code above.

Next time: Fabulous adventures\!

