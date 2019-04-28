<div id="page">

# What are threading models, and what threading model do the script engines use?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 5:28:00 PM

-----

<div id="content">

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I've got a few ideas for some future posts that depend on the reader understanding a little bit about COM threading.<span style="mso-spacerun: yes">  </span>Since I myself understand only a little bit about COM threading, I'll just do a brain dump for you all right here.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I'm sure you all know about multi-threaded applications.<span style="mso-spacerun: yes">  </span>The idea is that the operating system switches back and forth between threads within a process using a scheduling algorithm of some sort.<span style="mso-spacerun: yes">  </span>When a thread is "frozen" all the context for that thread -- basically, the values of the registers in the processor -- is saved, and when it is "thawed" the state is restored and the thread continues like it was never interrupted.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">That works great right up to the point where two threads try to access the same memory at the same time.<span style="mso-spacerun: yes">  </span>Consider, for example, the standard implementation of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IUnknown::Release</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">():</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">ULONG MyClass::Release()  
{</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>--this-\>m\_cRef;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>if (this-\>m\_cRef == 0)</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>{</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">    </span>delete this;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">    </span>return 0;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>}</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>return this-\>m\_cRef;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">}</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Now suppose the ref count is two and two threads try to each do a single release.<span style="mso-spacerun: yes">  </span>That should work just fine, right?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Wrong.<span style="mso-spacerun: yes">  </span>The problem is that though <span style="mso-spacerun: yes"> </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">--this-\>m\_cRef </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">looks like a single "atomic" operation, the compiler actually spits out code that acts something vaguely like this pseudo-code:</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>// "this" is stored in Register1</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>1 copy *address* of "m\_cRef" field of pointer stored in Register1 to Register2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>2 copy *contents* of address stored in Register2 to Register3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>3 decrease contents of Register3 by one</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>4 copy *contents* of Register3 back to *address* stored in Register2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>5 compare contents of Register3 to zero, store Boolean result of comparison in Register4</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>6 if Register4 is false then return contents of Register3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>7 ... etc -- do deletion, return zero</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Notice that the compiler can be smart and re-use the contents of Register3 instead of fetching </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">this-\>m\_cRef</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> three times.<span style="mso-spacerun: yes">  </span>The compiler knows that no one has changed it since the decrease.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So suppose we have a red thread and a blue thread.<span style="mso-spacerun: yes">  </span>Each has their own registers.<span style="mso-spacerun: yes">  </span>Suppose the processor schedules them in this order:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>1 copy address of "m\_cRef" field of Register1 to Register2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>2 copy contents of address stored in Register2 to Register3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">CONTEXT SWITCH\! Save <span style="mso-spacerun: yes">   </span>Register1 = this, Register2 = \&m\_cRef, Register3 = 2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: red; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>1 copy address of "m\_cRef" field of Register 1 to Register 2</span>

<span style="FONT-SIZE: 10pt; COLOR: red; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>2 copy contents of address stored in Register 2 to Register 3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: red; FONT-FAMILY: &#39;Lucida Console&#39;">CONTEXT SWITCH\! Save <span style="mso-spacerun: yes">   </span>Register1 = this, Register2 = \&m\_cRef, Register3 = 2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">CONTEXT SWITCH\! Restore Register1 = this, Register2 = \&m\_cRef, Register3 = 2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>3 decrease contents of Register3 by one</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>4 copy contents of Register3 to address stored in Register2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>5 compare contents of Register3 to zero, store Boolean result of comparison in Register4</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>6 if Register4 is false then return contents of Register3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Register 4 is false because Register3 = 1, so this returns 1.</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: red; FONT-FAMILY: &#39;Lucida Console&#39;">CONTEXT SWITCH\! Restore Register1 = this, Register2 = \&m\_cRef, Register3 = 2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And now you see where this is going, I'm sure.<span style="mso-spacerun: yes">  </span>**Because the original value was stored in the red thread before the blue thread decremented it, we've lost a decrement**. <span style="mso-spacerun: yes"> Both threads will return 1. T</span>his object's ref count will never go to zero, and its memory will leak.<span style="mso-spacerun: yes">  </span>A similar problem plagues </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">AddRef</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- you can lose increment operations, which causes memory to be freed too soon.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">How do we solve this problem?<span style="mso-spacerun: yes">  </span>Basically there are two ways to do it: </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">1)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Do the necessary work to ensure thread safety</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">2)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Require your callers to behave in a manner such that you never get into this situation in the first place.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The operating system provides tools to make multi-threaded programming work.<span style="mso-spacerun: yes">  </span>There are methods like </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">InterlockedIncrement</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, which really do "atomically" bump up a counter. Signals and semaphores and critical sections and all the other tools you need to make multi-threaded programs are available. I'm not going to talk much about those. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Writing a truly free-threaded program is a lot of work.<span style="mso-spacerun: yes">  </span>There are a lot of ways to get it wrong, and there are potential performance pitfalls as well.<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Fortunately, there is a middle ground between "only one thread allowed" and "any thread can call any method at any time".<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>The idea of the COM threading models is to provide a **contract** between callers and callees so that, as long as both sides follow the contract, situations like the one above never come to pass.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Suppose a caller has several instances of an object (the callee), and the caller has several threads going.<span style="mso-spacerun: yes">  </span>The commonly used standard threading contracts are as follows:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* **Single threaded** -- all calls to all instances of the object must always be on the same thread.<span style="mso-spacerun: yes">  </span>There are no synchronization issues because there is always only one thread no matter how many object instances there are.<span style="mso-spacerun: yes">  </span>The **caller** is responsible for ensuring that **all calls to all instances are on the same thread**.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* **Free threaded** -- calls to the object can be on **any thread** at **any time**, including multiple threads at the same time.<span style="mso-spacerun: yes">  </span>The **object** is responsible for **all synchronization issues**.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* **Apartment threaded** -- all calls to any **given instance** of the object must always be on the same thread, but **different instances can be called on different threads at the same time**.<span style="mso-spacerun: yes">  </span>The **caller** is responsible for ensuring that **given an instance, all calls to that instance happen on the same thread**.<span style="mso-spacerun: yes">  </span>The **object** is responsible for synchronizing access to global (that is, not-per-instance) data that it owns.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>An analogy might help. Think of an apartment building where each apartment is a thread and each person is an object instance.<span style="mso-spacerun: yes">  </span>You can put as many people into one apartment as you want, and you can put people into lots of different apartments, but once you've done so, **you always have to go to a person's apartment if you want to talk to them**.<span style="mso-spacerun: yes">  Why? Because they never move out once they're in an apartment, you have to wait for them to die before they ever leave.  (Insert New Yorker joke here.)  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>Furthermore, you can't talk "through the walls" from one apartment to someone in another apartment.<span style="mso-spacerun: yes">  (Well, actually you can -- that's called "marshaling", and that's a subject for a future post.)  And finally, i</span>f the people jointly own a shared resource -- say, a rooftop barbecue, to stretch this silly analogy to its limit -- then they must sort out amongst themselves how to synchronize access to the shared resource.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* **Rental threaded** -- calls to an object can be on **any thread** but the **caller guarantees that only one thread is calling into the object at any time**.<span style="mso-spacerun: yes">  </span>Rental threading requires a different analogy: suppose the object instances are rented televisions and again  threads are apartments.<span style="mso-spacerun: yes">  A television can be moved from apartment to apartment but can never be in more than one apartment at the same time. Multiple televisions can be in the same apartment, and multiple apartments can have multiple televisions.  **But if you want to watch a television, you have to go to the apartment where it is.**</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Whew, that was a long preamble.<span style="mso-spacerun: yes">  </span>How does this pertain to the script engines?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Most COM objects -- almost all ActiveX objects, and all of the object models commonly used by script -- are apartment threaded objects.<span style="mso-spacerun: yes">  </span>They expect that multiple instances of the object can be created on multiple threads, but once an instance is created on a thread, it will always be called on that thread.<span style="mso-spacerun: yes">  </span>This gives us the best of both worlds -- the caller can be free threaded and can create multiple objects on multiple threads, but the callee does not have to synchronize access to any per-instance data.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But the script engines are free threaded objects. **The script engines must ensure that they do not violate the apartment model contract.**<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So guess what?<span style="mso-spacerun: yes">  </span>The script engines actually have a bizarre, custom contract that is a little more restrictive than free threading and less restrictive than apartment threading\!<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The script engine contract is as follows:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* When the script engine is in a state where it cannot possibly call an ActiveX object -- for instance, if it has just been created and has not started running code, or if it is just about to be shut down -- then the script engine really is free threaded, but who cares? <span style="mso-spacerun: yes"> </span>It can't do much in this state.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* When the script engine is initialized -- when the script engine host has started the process of passing code and object model state to the engine -- the script engine morphs into an apartment threaded object.<span style="mso-spacerun: yes">  </span>All calls to the script engine must be on the initializing thread until the script engine is shut down again.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* There are two exceptions to the previous rule -- the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">InterruptScriptThread <span style="font-family: Lucida Sans Unicode; color: #800080;">and </span>Clone</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> methods can always be called from any thread.<span style="mso-spacerun: yes">  </span>The former is the mechanism whereby the host can signal a lo</span>

</div>

</div>

