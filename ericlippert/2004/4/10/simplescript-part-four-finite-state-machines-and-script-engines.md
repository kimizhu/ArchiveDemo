<div id="page">

# SimpleScript Part Four: Finite State Machines and Script Engines

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/10/2004 5:18:00 PM

-----

<div id="content">

<span>[Last time](/ericlippert/archive/2004/04/05/108086.aspx "http://blogs.msdn.com/ericlippert/archive/2004/04/05/108086.aspx") I said that I'd discuss finite state machines (also sometimes called finite state automata, same thing.)  The FSM is a fundamental idea in theoretical computer science because it models computing machinery in a very simple, abstract and general way.  Basically it goes like this:  an FSM has a **<span>finite</span>** number of **<span>states</span>** (duh).  Each state accepts a finite number of **<span>inputs</span>**.  Each state has **<span>rules</span>** which describe the **<span>action</span>** of the machine for every input.  An input may cause the machine to change state. </span>

<span></span>

<span>That's extremely general, I know.  Let's consider a specific example -- a machine which has two inputs: a slot that can take in quarters, and a button.  It has two outputs: quarters and gumballs.  And it has two states: GOTQUARTER and NOQUARTER.  We can chart the actions given a certain input: </span>

<span>  </span>

<span> </span>

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
<td><p><span>GOTQUARTER </span></p></td>
<td><p><span>Output the quarter </span></p></td>
<td><p><span>Output gumball, </span></p>
<p><span>Change state to NOQUARTER</span></p></td>
</tr>
</tbody>
</table>

<span></span>

<span>That's a pretty darn simple machine.  Modern CPU's are also finite state machines -- after all, every bit in a machine can only be in two states, and there are only so many possible binary inputs from mice, keyboards, hard disks, and so on.  The number of possible states a modern computer can be in is mind-bogglingly huge -- far more than the total number of particles in the universe -- but because the rules for the state transitions are very well defined it all works out nicely.  (Because the number of states is so large, it is often more helpful to model machines as "Turing machines", which can have unlimited internal storage.  But I'm digressing.) </span>

<span></span>

<span>The script engines use this idea of finite state machines explicitly.  A script engine has six possible states, and it performs certain actions as it transitions between those states based on "inputs".  In this case, the inputs are via calls to the various methods on the script engine.  The six states are **<span>Uninitialized</span>**, **<span>Initialized</span>**, **<span>Started</span>**, **<span>Connected</span>**, **<span>Disconnected</span>** and **<span>Closed</span>**.  A script engine is created by a host -- Windows Script Host, Internet Explorer, Active Server Pages, whatever.  The host is responsible for changing the script engine state as it sees fit, within the rules I'll describe over the next few entries. </span>

<span></span>

<span>The script engine begins life **<span>Uninitialized</span>**.  An uninitialized engine does not know anything about its host, and therefore cannot call any methods provided by the host.  There's very little you can do to an uninitialized engine, but what little you can do, you can do from any thread, subject to the rental model.  I'll talk more about threading issues later. </span>

<span></span>

<span>There's only one way to initialize an engine, and that's to pass a pointer to the host into </span><span>SetScriptSite</span><span>.  Once an engine is **<span>Initialized</span>** it becomes apartment threaded, with a few exceptions that I'll talk about later.  There is a lot more you can do with an initialized engine -- add named items (again, more on those in the future), add script source code, and so on.  However, the code cannot actually run yet.  At this point, the most commonly used input that affects the script engine state is the appropriately named </span><span>SetScriptState</span><span> method.  More on that below. </span>

<span></span>

<span>A **<span>Started</span>** engine actually runs the script code, but does not hook up event handlers.  (I discussed implicit event binding [last time](/ericlippert/archive/2004/04/05/108086.aspx "http://blogs.msdn.com/ericlippert/archive/2004/04/05/108086.aspx").)  Moving the engine into **<span>Connected</span>** state hooks up the event handlers.  If the host wants the event handlers to temporarily stop handling events, the hose can move the engine into **<span>Disconnected</span>** state. </span>

<span></span>

<span>That's the usual order in which things happen at startup.  To tear down the engine, the </span><span>IActiveScript::Close</span><span> method tears down event handlers, throws away compiled state, throws away the reference to the host and moves the engine to **<span>Closed</span>** state.  </span>

<span></span>

<span>The actions associated with </span><span>SetScriptState</span><span> are easiest to summarize in a table.  (Except for illegal calls, this also sets the engine state to the new value.) </span>

<span>  </span>

<span></span>

<span>Old State</span>

</div>

</div>

<span>New State passed to SetScriptState</span>

<span></span>

<span>Uninitialized</span>

<span>Initialized</span>

<span>Started</span>

<span>Connected</span>

<span>Disconnected</span>

<span>Closed</span>

<span>Uninitialized</span>

<span>Do nothing</span>

<span>Error -- use SetScriptSite</span>

<span>Error - can't start an uninitialized engine</span>

<span>Error </span>

<span>Error -- can't disconnect an unconnected engine</span>

<span>Error </span>

<span>Initialized</span>

<span>Discard some named items </span>

<span>Throw away host information</span>

<span>Do nothing </span>

<span>Execute script code </span>

<span>Execute script code </span>

<span>Bind events</span>

<span>Error</span>

<span>Discard all named items and script blocks </span>

<span>Throw away host information</span>

<span>Started</span>

<span>Discard some script blocks, and as above</span>

<span>Discard some script blocks</span>

<span>Do nothing </span>

<span>Bind Events </span>

<span>Error </span>

<span>As above </span>

<span>Connected</span>

<span>Discard event bindings, and as above</span>

<span>Discard event bindings, and as above</span>

<span>Discard event bindings, execute code</span>

<span>Do nothing</span>

<span>Suspend event handling</span>

<span>Discard event bindings, and as above</span>

<span>Disconnected</span>

<span>As above</span>

<span>As above</span>

<span>As above</span>

<span>Resume event handling</span>

<span>Do nothing</span>

<span>As above</span>

<span>Closed</span>

<span>Error</span>

<span>Error -- use SetScriptSite</span>

<span>Error</span>

<span>Error</span>

<span>Error</span>

<span>Do nothing</span>

<span></span>

<span>As you can see, I've written methods that implement these state semantics in [engine.cpp](/ericlippert/articles/108025.aspx "http://blogs.msdn.com/ericlippert/articles/108025.aspx"), though plenty of them are still stubbed out. </span>

<span></span>

<span>Next time, I want to talk more about the threading model, explain away a few oddities, and then we'll change gears for a bit and implement a simple language.  That'll get us into language design, how to expose syntax errors, and so on.</span>

