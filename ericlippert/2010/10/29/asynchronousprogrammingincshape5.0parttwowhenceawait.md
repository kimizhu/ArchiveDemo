# Asynchronous Programming in C\# 5.0 part two: Whence await?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/29/2010 6:44:00 AM

-----

I want to start by being absolutely positively clear about two things, because our usability research has shown this to be confusing. Remember our little program from last time? async void ArchiveDocuments(List\<Url\> urls)  
{  
  Task archive = null;  
  for(int i = 0; i \< urls.Count; ++i)  
  {  
    var document = await FetchAsync(urls\[i\]);  
    if (archive \!= null)  
      await archive;  
    archive = ArchiveAsync(document);  
  }  
}

The two things are:

**1)** The “async” modifier on the method does **not** mean “*this method is automatically scheduled to run on a worker thread asynchronously*”. It means the **opposite** of that; it means “*this method contains control flow that involves awaiting asynchronous operations and will therefore be rewritten by the compiler into continuation passing style to ensure that the asynchronous operations can resume this method at the right spot*.” **The whole point of async methods it that you stay on the current thread as much as possible.** They’re like coroutines: async methods bring single-threaded cooperative multitasking to C\#. (At a later date I’ll discuss the reasons behind requiring the async modifier rather than inferring it.)****

2\) The “await” operator used twice in that method does **not** mean “*this method now blocks the current thread until the asynchronous operation returns*”. That would be making the asynchronous operation back into a synchronous operation, which is precisely what we are attempting to avoid. Rather, it means the **opposite** of that; it means “*if the task we are awaiting has not yet completed then sign up the rest of this method as the continuation of that task, and then return to your caller immediately; the task will invoke the continuation when it completes.*”

It is unfortunate that people’s intuition upon first exposure regarding what the “async” and “await” contextual keywords mean is frequently the opposite of their actual meanings. Many attempts to come up with better keywords failed to find anything better. If you have ideas for a keyword or combination of keywords that is short, snappy, and gets across the correct ideas, I am happy to hear them. Some ideas that we already had and rejected for various reasons were:

wait for FetchAsync(…)  
yield with FetchAsync(…)  
yield FetchAsync(…)  
while away the time FetchAsync(…)  
hearken unto FetchAsync(…)  
for sooth Romeo wherefore art thou FetchAsync(…)

Moving on. We’ve got a lot of ground to cover. The next thing I want to talk about is “what exactly are those ‘thingies’ that I handwaved about last time?”

Last time I implied that the C\# 5.0 expression document = await FetchAsync(urls\[i\])

gets realized as: state = State.AfterFetch;  
fetchThingy = FetchAsync(urls\[i\]);  
if (fetchThingy.SetContinuation(archiveDocuments))  
  return;  
AfterFetch: ;  
document = fetchThingy.GetValue();

what’s the thingy?

In our model for asynchrony an asynchronous method typically returns a Task\<T\>; let’s assume for now that FetchAsync returns a Task\<Document\>. (Again, I’ll discuss the reasons behind this "Task-based Asynchrony Pattern" at a later date.) The actual code will be realized as:

fetchAwaiter = FetchAsync(urls\[i\]).GetAwaiter();  
state = State.AfterFetch;  
if (fetchAwaiter.BeginAwait(archiveDocuments))  
  return;  
AfterFetch: ;  
document = fetchAwaiter.EndAwait();

The call to FetchAsync creates and returns a Task\<Document\> - that is, an object which represents a “hot” running task. Calling this method *immediately* returns a Task\<Document\> which is then somehow asynchronously fetches the desired document. Perhaps it runs on another thread, or perhaps it posts itself to some Windows message queue on this thread that some message loop is polling for information about work that needs to be done in idle time, or whatever. That’s its business. What we know is that we need something to happen when it completes. (Again, I’ll discuss single-threaded asynchrony at a later date.)

To make something happen when it completes, we ask the task for an Awaiter, which exposes two methods. BeginAwait signs up a continuation for this task; when the task completes, a miracle happens: somehow the continuation gets called. (Again, how exactly this is orchestrated is a subject for another day.) If BeginAwait returns true then the continuation will be called; if not, then that’s because the task has already completed and there is no need to use the continuation mechanism.

EndAwait extracts the result that was the result of the completed task.

We will provide implementations of BeginAwait and EndAwait on Task (for tasks that are logically void returning) and Task\<T\> (for tasks that return a value). But what about asynchronous methods that do not return a Task or Task\<T\> object? Here we’re going to use the same strategy we used for LINQ. In LINQ if you say from c in customers where c.City == "London" blah blah blah 

then that gets translated into

customers.Where(c=\>c.City=="London") …

and overload resolution tries to find the best possible Where method by checking to see if customers implements such a method, or, if not, by going to extension methods. The GetAwaiter / BeginAwait / EndAwait pattern will be the same; we’ll just do overload resolution on the transformed expression and see what it comes up with. If we need to go to extension methods, we will.

Finally: why "Task"?

The insight here is that asynchrony does not require parallelism, but parallelism does require asynchrony, and many of the tools useful for parallelism can be used just as easily for non-parallel asynchrony. There is no inherent parallelism in Task; that the Task Parallel Library uses a task-based pattern to represent units of pending work that can be parallelized does not *require* multithreading.

As I've pointed out a few times, from the point of view of the code that is waiting for a result it really doesn't matter whether that result is being computed in idle time on this thread, in a worker thread in this process, in another process on this machine, on a storage device, or on a machine halfway around the world. What matters is that it's going to take time to compute the result, and this CPU could be doing something else while it is waiting, if only we let it.

The Task class from the TPL already has a lot of investment in it; it's got a cancellation mechanism and other useful features. Rather than invent some new thing, like some new "IFuture" type, we can just extend the existing task-based code to meet our asynchrony needs.

Next time: How to further compose asynchronous tasks.

