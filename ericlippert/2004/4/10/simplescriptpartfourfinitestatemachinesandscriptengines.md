# SimpleScript Part Four: Finite State Machines and Script Engines

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/10/2004 5:18:00 PM

-----

[Last time](/ericlippert/archive/2004/04/05/108086.aspx "http://blogs.msdn.com/ericlippert/archive/2004/04/05/108086.aspx") I said that I'd discuss finite state machines (also sometimes called finite state automata, same thing.)  The FSM is a fundamental idea in theoretical computer science because it models computing machinery in a very simple, abstract and general way.  Basically it goes like this:  an FSM has a **finite** number of **states** (duh).  Each state accepts a finite number of **inputs**.  Each state has **rules** which describe the **action** of the machine for every input.  An input may cause the machine to change state. 

That's extremely general, I know.  Let's consider a specific example -- a machine which has two inputs: a slot that can take in quarters, and a button.  It has two outputs: quarters and gumballs.  And it has two states: GOTQUARTER and NOQUARTER.  We can chart the actions given a certain input: 

  

 

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><span></span></p></td>
<td><p><span>Insert Quarter</span></p></td>
<td><p><span>Press Button</span></p></td>
</tr>
<tr class="even">
<td><p><span>NOQUARTER</span></p></td>
<td><p><span>Change state to GOTQUARTER</span></p></td>
<td><p><span>Do nothing</span></p></td>
</tr>
<tr class="odd">
<td><p><span>GOTQUARTER </span></p>
<p></p>
<p></p></td>
<td><p><span>Output the quarter </span></p>
<p></p>
<p></p></td>
<td><p><span>Output gumball, </span></p>
<p></p>
<p></p>
<p><span>Change state to NOQUARTER</span></p></td>
</tr>
</tbody>
</table>

That's a pretty darn simple machine.  Modern CPU's are also finite state machines -- after all, every bit in a machine can only be in two states, and there are only so many possible binary inputs from mice, keyboards, hard disks, and so on.  The number of possible states a modern computer can be in is mind-bogglingly huge -- far more than the total number of particles in the universe -- but because the rules for the state transitions are very well defined it all works out nicely.  (Because the number of states is so large, it is often more helpful to model machines as "Turing machines", which can have unlimited internal storage.  But I'm digressing.) 

The script engines use this idea of finite state machines explicitly.  A script engine has six possible states, and it performs certain actions as it transitions between those states based on "inputs".  In this case, the inputs are via calls to the various methods on the script engine.  The six states are **Uninitialized**, **Initialized**, **Started**, **Connected**, **Disconnected** and **Closed**.  A script engine is created by a host -- Windows Script Host, Internet Explorer, Active Server Pages, whatever.  The host is responsible for changing the script engine state as it sees fit, within the rules I'll describe over the next few entries. 

The script engine begins life **Uninitialized**.  An uninitialized engine does not know anything about its host, and therefore cannot call any methods provided by the host.  There's very little you can do to an uninitialized engine, but what little you can do, you can do from any thread, subject to the rental model.  I'll talk more about threading issues later. 

There's only one way to initialize an engine, and that's to pass a pointer to the host into SetScriptSite.  Once an engine is **Initialized** it becomes apartment threaded, with a few exceptions that I'll talk about later.  There is a lot more you can do with an initialized engine -- add named items (again, more on those in the future), add script source code, and so on.  However, the code cannot actually run yet.  At this point, the most commonly used input that affects the script engine state is the appropriately named SetScriptState method.  More on that below. 

A **Started** engine actually runs the script code, but does not hook up event handlers.  (I discussed implicit event binding [last time](/ericlippert/archive/2004/04/05/108086.aspx "http://blogs.msdn.com/ericlippert/archive/2004/04/05/108086.aspx").)  Moving the engine into **Connected** state hooks up the event handlers.  If the host wants the event handlers to temporarily stop handling events, the hose can move the engine into **Disconnected** state. 

That's the usual order in which things happen at startup.  To tear down the engine, the IActiveScript::Close method tears down event handlers, throws away compiled state, throws away the reference to the host and moves the engine to **Closed** state.  

The actions associated with SetScriptState are easiest to summarize in a table.  (Except for illegal calls, this also sets the engine state to the new value.) 

  

Old State

New State passed to SetScriptState

Uninitialized

Initialized

Started

Connected

Disconnected

Closed

Uninitialized

Do nothing

Error -- use SetScriptSite

Error - can't start an uninitialized engine

Error 

Error -- can't disconnect an unconnected engine

Error 

Initialized

Discard some named items 

Throw away host information

Do nothing 

Execute script code 

Execute script code 

Bind events

Error

Discard all named items and script blocks 

Throw away host information

Started

Discard some script blocks, and as above

Discard some script blocks

Do nothing 

Bind Events 

Error 

As above 

Connected

Discard event bindings, and as above

Discard event bindings, and as above

Discard event bindings, execute code

Do nothing

Suspend event handling

Discard event bindings, and as above

Disconnected

As above

As above

As above

Resume event handling

Do nothing

As above

Closed

Error

Error -- use SetScriptSite

Error

Error

Error

Do nothing

As you can see, I've written methods that implement these state semantics in [engine.cpp](/ericlippert/articles/108025.aspx "http://blogs.msdn.com/ericlippert/articles/108025.aspx"), though plenty of them are still stubbed out. 

Next time, I want to talk more about the threading model, explain away a few oddities, and then we'll change gears for a bit and implement a simple language.  That'll get us into language design, how to expose syntax errors, and so on.

