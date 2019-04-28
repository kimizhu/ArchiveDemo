# What is this thing you call "thread safe"?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/19/2009 6:50:00 AM

-----

**Caveat: I am not an expert on multi-threading programming.** In fact, I wouldn't even say that I am *competent* at it. My whole career, I've needed to write code to spin up a secondary worker thread probably less than half a dozen times. So take everything I say on the subject with some skepticism.

A question I'm frequently asked: "*is this code **thread safe**?*" To answer the question, clearly we need to know what "thread safe" means.

But before we get into that, there's something I want to clear up first. A question I am less frequently asked is "*Eric, why does Michelle Pfeiffer always look so good in photographs?*" To help answer this pressing question, I consulted [Wikipedia](http://en.wikipedia.org/wiki/Photogenic):

> *"A **photogenic** subject is a subject that usually appears physically attractive or striking in photographs."*

Why does Michelle Pfeiffer always look so good in photographs? ***Because she's photogenic*.** Obviously.

Well, I'm glad we've cleared up that mystery, but I seem to have wandered somehwat from the subject at hand. Wikipedia is [just as helpful in defining thread safety](http://en.wikipedia.org/wiki/Thread_safety):

> * "A piece of code is **thread-safe** if it functions correctly during simultaneous execution by multiple threads."*

As with photogenicity, this is obvious question-begging. When we ask "is this code *thread safe*?" all we are really asking is "is this code *correct* *when called in a particular manner*?" So how do we determine if the code is correct? **We haven't actually explained anything here.**

Wikipedia goes on:

> *"In particular, it must satisfy the need for multiple threads to access the same shared data, ..."*

This seems fair; this scenario is almost always what people mean when they talk about thread safety. But then:

> *"...and the need for a shared piece of data to be accessed by only one thread at any given time."*

Now we're talking about techniques for *creating* thread safety, not *defining* what thread safety means. Locking data so that it can only be accessed by one thread at a time is just one possible technique for creating thread safety; it is not itself the definition of thread safety.

My point is not that the definition is *wrong*; as informal definitions of thread safety go, this one is not terrible. Rather, my point is that the definition indicates that the concept itself is *completely vague* and essentially means nothing more than "behaves correctly in some situations". Therefore, when I'm asked "is this code thread safe?" I always have to push back and ask "what are the *exact* *threading scenarios* you are concerned about?" and "exactly what is *correct behaviour* of the object in every one of those scenarios?"

Communication problems arise when people with different answers to those questions try to communicate about thread safety. For example, suppose I told you that I have a "threadsafe mutable queue" that you can use in your program. You then cheerfully write the following code that runs on one thread while another thread is busy adding and removing items from the mutable queue:

 

if (\!queue.IsEmpty) Console.WriteLine(queue.Peek());

Your code then crashes when the Peek throws a QueueEmptyException. What is going on here? I said this thing was thread safe, and yet your code is crashing in a multi-threaded scenario.

When I said "the queue is threadsafe" I meant that the queue maintains its internal state consistently no matter what sequence of *individual* operations are happening on other threads. But I did not mean that you can use my queue in any scenario that requires *logical consistency maintained across multiple operations in a sequence*. In short, my opinion of "correct behaviour" and your opinion of the same differed because what we thought of as the relevant scenario was completely different. I care only about not crashing, but you care about being able to reason logically about the information returned from each method call.

In this example, you and I are probably talking about different kinds of thread safety. Thread safety of mutable data structures is usually all about ensuring that the operations on the shared data always operate on the **most up-to-date** state of the shared data as it mutates, even if that means that a particular combination of operations appears to be **logically inconsistent**, as in our example above. Thread safety of immutable data structures is all about ensuring that use of the data across all operations is **logically consistent**, at the expense of the fact that you're looking at an immutable snapshot that might be **out-of-date**.

The problem here is that the choice about whether to access the first element or not is based on "stale" data. Designing a truly thread-safe mutable data structure in a world where *nothing is allowed to be stale* can be very difficult. Consider what you'd have to do in order to make the "Peek" operation above actually threadsafe. You'd need a new method:

 

if (\!queue.Peek(out first)) Console.WriteLine(first);

Is this "thread safe"? It certainly seems better. But what if after the Peek, a different thread dequeues the queue? Now you're not crashing, but you've changed the behaviour of the previous program considerably. In the previous program, if, after the test there was a dequeue on another thread that changed what the first element was, then you'd either crash or print out the up-to-date first element in the queue. Now you're printing out a *stale* first element. Is that *correct*? Not if we *always* want to operate on up-to-date data\!

But wait a moment -- actually, the *previous* version of the code had this problem as well. What if the dequeue on the other thread happened *after* the call to Peek succeeded but *before* the Console.WriteLine call executed? Again, you could be printing out stale data.

What if you want to ensure that you are always printing out up-to-date data? What you really need to make this threadsafe is:

 

queue.DoSomethingToHead(first=\>{Console.WriteLine(first);});

Now the queue author and the queue user agree on what the relevant scenarios are, so this is truly threadsafe. Right?

Except... there could be something super-complicated in that delegate. What if whatever is in the delegate happens to cause an event that triggers code to run on another thread, which in turn causes some queue operation to run, which in turn blocks in such a manner that we've produced a deadlock? Is a deadlock "correct behaviour"? And if not, is this method truly "safe"?

Yuck.

By now you take my point I'm sure. As I pointed out earlier, [it is unhelpful to say that a building or a hunk of code is "secure" without somehow communicating **which threats** the utilized security mechanism are and are not proof against.](http://blogs.msdn.com/ericlippert/archive/2008/08/19/tasty-beverages.aspx) Similarly, **it is unhelpful to say that code is "thread safe" without somehow communicating what undesirable behaviors the utilized thread safety mechanisms do and do not prevent.** "Thread safety" is nothing more nor less than a code contract, like any other code contract. You agree to talk to an object in a particular manner, and it agrees to give you correct results if you do so; working out exactly what that manner is, and what the correct responses are, is a potentially tough problem.

\*\*\*\*\*\*\*\*\*\*\*\*

(\*) Yes, I'm aware that if I think something on Wikipedia is wrong, I can change it. There are two reasons why I should not do so. First, as I've already stated I'm not an expert in this area; I leave it to the experts to sort out amongst themselves what the right thing to say here is. And second, my point is not that the Wikipedia page is wrong, but rather that it illustrates that the term itself is vague by nature.

