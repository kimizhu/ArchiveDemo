# Asynchrony in C\# 5 Part Five: Too many tasks

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/8/2010 6:26:00 AM

-----

Suppose a city has a whole bunch of bank branches, each of which has a whole bunch of tellers and one gofer. There are a whole bunch of customers in the city, each of whom wants to withdraw a whole bunch of money from the bank at some varying time throughout the day.

The algorithm goes like this:

A customer finds the nearest bank branch and evaluates its line. If the line is out the door, then the customer goes to another bank branch. This continues until they either find one with a short enough line, or they give up and go home.

Suppose they find one with a short enough line. The customer queues up. ([Perhaps they queue up using the M model or using the W model](http://blogs.msdn.com/b/ericlippert/archive/2009/08/20/queueing-theory-in-action-plus-frogs.aspx); it doesn't particularly matter for the purpose of this analogy. Let's suppose its the W model.) When the customer reaches a teller, the transaction goes like this: the customer requests a certain number of bills of various denominations, and the teller counts them out: one, two, three, four, five, there you go. The customer leaves and the teller services the next customer.

This seems perfectly reasonable. But suppose the teller runs out of the right denomination of cash halfway through the transaction. The teller tells the gofer to go get more fifties out of the vault, and then has a little nap while the gofer is running to the vault and back. Now the customer, and **every customer behind them**, has to wait for the teller to wake up, which doesn't happen until the gofer gets back from the vault.

In case it's not clear, in this analogy a city is a server farm, a bank branch is a machine, a teller is a worker thread, the gofer is an I/O completion thread, and a customer is a client. The money is the computation that the server performs on behalf of the client, and waiting for the gofer to go to the vault is a synchronous delay waiting on I/O completion. Deciding whether the line is too long to choose a branch is load balancing the server farm.

Now, you might say that this could be more efficient. That time when the teller is asleep waiting for the gofer is troubling. Ideally you want one thread per core going full out all the time. The teller could be servicing the next customer in line, and then pick up with the first customer when the gofer gets back. This seems like an ideal use of task-based asynchrony.

You have to be really careful to understand the implications on the whole system when you make a change like this. It is really easy to accidentally implement the following system:

A customer finds the nearest bank branch. There is no line, so they go right in. There are no tellers. There's just a "take a number" machine and an arrow pointing to a door. The customer takes a number and walks through the door into an **enormous warehouse** containing tens of thousands of customers. There are a bunch of tellers running at top speed from customer to customer, giving them each **one bill at a time**. If a teller runs out of a needed denomination then they yell at the gofer, and keep right on going to the next customer who they can serve. Eventually the gofer brings them the needed denomination. The tellers keep track of who all they are servicing "at the same time", and no one leaves until they get all their money. If the gofer can't keep up then the warehouse just gets more and more crowded, and the more crowded it gets, the longer it takes for a customer to get all their money, and the more customers who arrive while they're waiting, and it just snowballs.

In this scheme the tellers are almost never asleep, so the CPU cores are being heavily utilized, and everyone gets their first bill reasonably fast. "Time to first byte of results" is potentially excellent. But the load balancing just went out the window; because there is never a line, there's no way for the load balancer to know that there are too many unserved customers. And time to last byte served can be potentially bad.

This sounds like a silly story, but we accidentally did something like this to ourselves just the other day. We built a code analyzer that uses task based asynchrony on the UI thread. The idea was that on every keystroke we would start an asynchronous task that then itself started other asynchronous tasks. The tasks would be :

1\) Check to see if there's been a subsequent keystroke recently; if so, then cancel any unfinished tasks associated with this keystroke.  
2\) Recolour the user interface text based on the differential analysis of the change to the syntax tree induced by the keystroke.  
3\) Set a timer to see if there has been half a second of time with no keystrokes. If so, the user may have paused typing and we can kick off a background worker to perform deeper program analysis.

See the problem? The tasks are created **so fast** between keystrokes that if you're typing quickly soon you end up with a warehouse full of tens of thousands of tasks, 99.99% of which have not yet run in order to cancel themselves. And half the ones that do manage to run are creating timers, the vast majority of which are going to be deleted before they ever tick. The garbage collection pressure from all those timers alone was enough to destroy performance. Asynchrony is awesome, but you need to make sure that you get the granularity of the asynchrony at the appropriate level. The code analyzer was rewritten to enqueue tasks that referred to a single, global timer, and to be less aggressive about enqueueing tasks that were highly likely to need cancellation a tenth of a second later. Performance improved drastically.

Task-based asynchrony is awesome but it is not a panacea; be careful and measure as you go. The fact that there is hardly any waiting between the time you make the request and the time the asynchronous request enqueues itself onto the work queue means that there is hardly any restriction on the number of tasks you can create quickly.

**Next time**: More thoughts on syntactic concerns

