# Asynchrony in C\# 5, Part Three: Composition

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/1/2010 6:34:00 AM

-----

I was walking to my bus the other morning at about 6:45 AM. Just as I was about to turn onto 45th street, a young man, shirtless, **covered in blood** ran down 45th at considerable speed right in front of me. Behind him was another fellow, wielding a baseball bat. My initial thought was "holy goodness, I have to call the police right now\!"

Then I saw that the guy with the baseball bat was himself being chased by Count Dracula, a small horde of zombies, a band of pirates, one medieval knight, and bringing up the rear, a giant bumblebee. Apparently some Wallingford jogging club got into the Hallowe'en spirit this past weekend.

In unrelated news: we've had lots of great comments and feedback on the async feature so far; **please keep it coming**. It is being read; it will take weeks to digest it and volume of correspondence may preclude personal replies, for which I apologize, but that's how it goes.

Today I want to talk a bit about composition of asynchronous code, why it is difficult in CPS, and how it is much easier with "await" in C\# 5.

I need a name for this thing. We named **LINQ** because it was **Language Integrated Query**. For now, let's provisionally call this new feature **TAP**, the **Task Asynchrony Pattern**. I'm sure we'll come up with a better name later; remember, this is still just a prototype. (\*)

The example I've been using so far, of fetching and archiving documents was obviously deliberately contrived to be a simple orchestration of two asynchronous tasks in a void returning method. As we saw, when using regular Continuation Passing Style it can be tricky to orchestrate even two asynchronous methods. Today I want to talk a bit about **composition** of asynchronous methods. Our ArchiveDocuments method was void returning, which simplifies things greatly. Suppose that ArchiveDocuments was to return a value, say, the total number of bytes archived. Synchronously, that's straightforward:

 

long ArchiveDocuments(List\<Url\> urls)  
{  
    long count = 0;  
    for(int i = 0; i \< urls.Count; ++i)  
    {  
        var document = Fetch(urls\[i\]);  
        count += document.Length;  
        Archive(document);  
    }  
    return count;  
}

Now consider how we'd rewrite that asynchronously. If ArchiveDocuments is going to return immediately when the first FetchAsync starts up, and is only going to be resumed when the first fetch completes, then when is the "return count;" going to be executed? The naive asynchronous version of ArchiveDocuments cannot return a count; it has to be written in CPS too:

 

void ArchiveDocumentsAsync(List\<Url\> urls, Action\<long\> continuation)  
{  
  // somehow do the archiving asynchronously,  
  // then call the continuation  
}

And now the taint has spread. Now the *caller* of ArchiveDocumentsAsync needs to be written in CPS, so that its continuation can be passed in. What if it in turn returns a result? This is going to become a mess; soon the entire program will be written upside down and inside out.

In the TAP model, instead we'd say that the type that represents asynchronous work that produces a result later is a Task\<T\>. In C\# 5 you can simply say:

 

async Task\<long\> ArchiveDocumentsAsync(List\<Url\> urls)  
{  
  long count = 0;  
  Task archive = null;  
  for(int i = 0; i \< urls.Count; ++i)  
  {  
    var document = await FetchAsync(urls\[i\]);  
    count += document.Length;  
    if (archive \!= null)  
      await archive;  
    archive = ArchiveAsync(document);  
  }  
  return count;  
}

and the compiler will take care of all the rewrites for you. It is instructive to understand exactly what happens here. This will get expanded into something like:

 

Task\<long\> ArchiveDocuments(List\<Url\> urls)  
{  
  var taskBuilder = AsyncMethodBuilder\<long\>.Create();  
  State state = State.Start;  
  TaskAwaiter\<Document\> fetchAwaiter = null;  
  TaskAwaiter archiveAwaiter = null;  
  int i;  
  long count = 0;  
  Task archive = null;  
  Document document;  
  Action archiveDocuments = () =\>  
  {  
    switch(state)  
    {  
      case State.Start:        goto Start;  
      case State.AfterFetch:   goto AfterFetch;  
      case State.AfterArchive: goto AfterArchive;  
    }  
    Start:  
    for(i = 0; i \< urls.Count; ++i)  
    {  
      fetchAwaiter = FetchAsync(urls\[i\]).GetAwaiter();  
      state = State.AfterFetch;  
      if (fetchAwaiter.BeginAwait(archiveDocuments))  
        return;  
      AfterFetch:  
      document = fetchAwaiter.EndAwait();  
      count += document.Length;  
      if (archive \!= null)  
      {  
        archiveAwaiter = archive.GetAwaiter();  
        state = State.AfterArchive;  
        if (archiveAwaiter.BeginAwait(archiveDocuments))  
          return;  
        AfterArchive:  
        archiveAwaiter.EndAwait();  
      }  
      archive = ArchiveAsync(document);  
    }  
    taskBuilder.SetResult(count);  
    return;  
  };  
  archiveDocuments();  
  return taskBuilder.Task;  
}

(Note that we still have problems with the labels being out of scope. Remember, the *compiler* doesn't need to follow the rules of C\# source code when it is generating code on your behalf; pretend the labels are in scope at the point of the goto. And note that we still have no **exception handling** in here. As I discussed last week in my post on building exception handling in CPS, exceptions get a little bit weird because there are \*two\* continuations: the normal continuation and the error continuation. How do we deal with that situation? I'll discuss how exception handling works in TAP at a later date.)

Let me make sure the control flow is clear here. Let's first consider the trivial case: the list is empty. What happens?  We create a task builder. We create a void-returning delegate. We invoke the delegate synchronously. It initializes the outer variable "count" to zero, branches to the "Start" label, skips the loop, tells the helper "you have a result", and returns. The delegate is now done. The taskbuilder is asked for a task; it knows that the task's work is completed, so it returns a *completed* task that simply represents the number zero.

If the caller attempts to await that task then its awaiter will return false when asked to begin async operations, because the task has completed. If the caller does not await that task then... well, then they do whatever they do with a Task. Eventually they can ask it for its result, or ignore it if they don't care.

Now let's consider the non-trivial case; there are multiple documents to archive. Again, we create a task builder and a delegate which is invoked synchronously. First time through the loop, we begin an asynchronous fetch, sign up the delegate as its continuation, and return from the delegate. At that point the task builder builds a task that represents "I'm asynchronously working on the body of ArchiveDocumentsAsync" and returns that task. When the fetch task completes asynchronously and invokes its continuation, the delegate starts up again "from the point where it left off" thanks to the magic of the state machine. Everything proceeds exactly as before, in the case of the void returning version; the only difference is that the returned Task\<long\> for ArchiveDocumentsAsync signals that it is complete (by invoking its continuation) when the delegate tells the task builder to set the result.

Make sense?

Before I continue with some additional thoughts on composition of tasks, a quick note on the **extensibility** of TAP. We designed LINQ to be very extensible; any type that implements Select, Where, and so on, or has extension methods implemented for them, can be used in query comprehensions. Similarly with TAP: any type that has a GetAwaiter that returns a type that has BeginAwait, EndAwait, and so on, can be used in "await" expressions. However, methods marked as being async can only return void, Task, or Task\<T\> for some T. We are all about enabling extensibility on **consumption** of existing asynchronous things, but have no desire to get in the business of enabling **production** of asynchronous methods with exotic types. (The alert reader will have noted that I have not discussed extensibility points for the **task builder**. At a later date I'll discuss where the task builder comes from.)

Continuing on: (ha ha ha)

In LINQ there are some situations in which the use of "language" features like "where" clauses is more natural and some where using "fluent" syntax ("Where(c=\>...)") is more natural. Similarly with TAP: our goal is to enable use of regular C\# syntax to compose and orchestrate asynchronous tasks, but sometimes you want to have a more "combinator" based approach. To that end, we'll be making available methods with names like like "WhenAll" or "WhenAny" that compose tasks like this:

 

    List\<List\<Url\>\> groupsOfUrls = whatever;  
    Task\<long\[\]\> allResults = Task.WhenAll(from urls in groupsOfUrls select ArchiveDocumentsAsync(urls));  
    long\[\] results = await allResults;

What does this do? Well, ArchiveDocumentsAsync returns a Task\<long\>, so the query returns an IEnumerable\<Task\<long\>\>. WhenAll takes a sequence of tasks and produces a new task which asynchronously awaits each of them, fills the result into an array, and then invokes its continuation with the results when available.

Similarly we'll have a WhenAny that takes a sequence of tasks and produces a new task that invokes its continuation with the first result when any of those tasks complete. (An interesting question is what happens if the first one completes successfully and the rest all throw an exception, but we'll talk about that later.)

There will be other task combinators and related helper methods; see the CTP samples for some examples. Note that in the CTP release we were unable to modify the existing Task class; instead we've provisionally added the new combinators to **TaskEx**. In the final release they will almost certainly be moved onto Task.

**Next time**: No, *seriously*, asynchrony does *not* necessarily involve multithreading.

(\*) I emphasize that this is **provisional** and **for my own rhetorical purposes,** and **not an official name of anything**. Please don't publish a book called "Essential TAP" or "Learn TAP in 21 Time Units" or whatever. I have a copy of "Instant DHTML Scriptlets" on my bookshelf; Dino Esposito writes so fast that he published an entire book between the time we mentioned the code name of the product to him and we announced the real name. ("Windows Script Components" was the final name.)

