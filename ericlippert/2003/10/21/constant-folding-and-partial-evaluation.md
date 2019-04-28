<div id="page">

# Constant Folding and Partial Evaluation

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/21/2003 1:31:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">A reader asks "*is there any reason why VBScript doesn't change* </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">str = str & "1234567890" & "hello"</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> <span style="mso-spacerun: yes"> </span>*to* </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">str = str & "1234567890hello"</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> *since they are both constants?*" </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Good question.<span style="mso-spacerun: yes">  </span>Yes, there are reasons. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The operation you're describing is called **constant folding**, and it is a very common compile-time optimization.<span style="mso-spacerun: yes">  </span>VBScript does an *extremely* limited kind of constant folding.<span style="mso-spacerun: yes">  </span>In VBScript, these two programs generate exactly the same code at the call site:</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const foo = "hello"</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print foo</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">is exactly the same as </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print "hello"</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">  
  
</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">That is, the code generated for both says "pass the **literal string** </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">"hello"</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> to the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> subroutine".<span style="mso-spacerun: yes">  </span>If </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> had been a variable instead of a constant then the code would have been generated to say "pass the **contents** of variable </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">..."<span style="mso-spacerun: yes">  </span>But the VBScript code generator is smart enough to realize that </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> is a constant, and so it does not generate a by-name or by-index lookup, it just slams the constant right in there so that there is no lookup indirection at all.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The kind of constant folding you're describing is *compile-time evaluation of expressions which have all operands known at compile time*.<span style="mso-spacerun: yes">  </span>For short, let's call it **partial evaluation**.<span style="mso-spacerun: yes">  </span>In C++ for example, it is legal to say</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const int CallMethod = 0x1;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const int CallProperty = 0x2;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const int CallMethodOrProperty = CallMethod | CallProperty;</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The C++ compiler is smart enough to realize that it can compute the third value itself.<span style="mso-spacerun: yes">  </span>VBScript would produce an error in this situation, as the compiler is not that smart.<span style="mso-spacerun: yes">  </span>Neither VBScript nor JScript will evaluate constant expressions at compile time.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">An even more advanced form of constant folding is to determine which functions are pure functions -- that is, functions which have no side effects, where the output of the function depends solely on the arguments passed in.<span style="mso-spacerun: yes">  </span>For example, in a language that supported pure functions, this would be legal:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const Real Pi = 3.14159265358979;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">const Real Sine60 = sine( Pi / 3);<span style="mso-spacerun: yes">  </span>// Pi / 3 radians = 60 degrees</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The sine function is a pure function -- there's no reason that it could not be called at compile time to assign to this constant.<span style="mso-spacerun: yes">  </span>However, in practice it can be very difficult to identify pure functions, and even if you can, there are issues in calling arbitrary code at compile time -- like, what if the pure function takes an hour to run?<span style="mso-spacerun: yes">  </span>That's a long compile\!<span style="mso-spacerun: yes">  </span>What if it throws exceptions?<span style="mso-spacerun: yes">  </span>There are many practical problems.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The JScript .NET compiler does support partial evaluation, but not pure functions.<span style="mso-spacerun: yes">  </span>The JScript .NET compiler architecture is quite interesting.<span style="mso-spacerun: yes">  </span>The source code is lexed into a stream of tokens, and then the tokens are parsed to form a parse tree.<span style="mso-spacerun: yes">  </span>Each node in the parse tree is represented by an object (written in C\#) which implements three methods: </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Evaluate</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">PartialEvaluate</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">TranslateToIL</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">. <span style="mso-spacerun: yes"> </span>When you call </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">PartialEvaluate</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on the root of the parse tree, it recursively descends through the tree looking for nodes representing operations where all the sub-nodes are known at compile time.<span style="mso-spacerun: yes">  </span>Those nodes are evaluated and collapsed into simpler nodes.<span style="mso-spacerun: yes">  </span>Once the tree has been evaluated as much as is possible at compile time, we then call </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">TranslateToIL</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, which starts another recursive descent that emits the IL into the generated assembly.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Evaluate</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> method is there to implement the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> function.<span style="mso-spacerun: yes">  </span>JScript Classic (which everyone thinks is an "interpreted" language) *always* compiles the script to bytecode and then interprets the bytecode -- even </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> calls the bytecode compiler in JScript Classic.<span style="mso-spacerun: yes">  </span>But in JScript Classic, a bytecode block is a block of memory entirely under control of the JScript engine, which can release it when the code is no longer callable.<span style="mso-spacerun: yes">  </span>In JScript .NET, we compile to IL which is then jitted into machine code.<span style="mso-spacerun: yes">  </span>If JScript .NET's implementation of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> emitted IL, then that jitted code would stay in memory until the appdomain went away\!<span style="mso-spacerun: yes">  </span>This means that a tight loop with an </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> in it is essentially a **memory leak** in JScript .NET, but not in JScript Classic.<span style="mso-spacerun: yes">  </span>Therefore, JScript .NET actually implements a true interpreter\!<span style="mso-spacerun: yes">  </span>In JScript .NET, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> generates a parse tree and does a full recursive evaluation on it.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I'm digressing slightly.<span style="mso-spacerun: yes">  </span>You wanted to know why the script engines don't implement partial evaluation.<span style="mso-spacerun: yes">  </span>Well, first of all, implementing partial evaluation would have made the script engines considerably more complicated for very little performance gain.<span style="mso-spacerun: yes">  </span>And if the author does want this gain, then the author can easily fold the constants "by hand".<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But more important, partial evaluation makes the process of compiling the script into bytecode much, much longer as you need to do yet another complete recursive pass over the parse tree.<span style="mso-spacerun: yes">  </span>That's great, isn't it?<span style="mso-spacerun: yes">  </span>I mean, that's trading increased compilation time for decreased run time.<span style="mso-spacerun: yes">  </span>What could be wrong with that? <span style="mso-spacerun: yes"> </span>Well, it depends who you ask.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">From the ASP implementers' perspective, that would indeed be great.<span style="mso-spacerun: yes">  </span>An ASP page, as I've already discussed, only gets compiled once, on the first page hit, but might be run many times.<span style="mso-spacerun: yes">  </span>Who cares if the **first** page hit takes a few milliseconds longer to do the compilation, if the subsequent million page hits each run a few microseconds faster?<span style="mso-spacerun: yes">  </span>And so what if this makes the VBScript DLL larger?<span style="mso-spacerun: yes">  </span>ASP updates are distributed to people with fast internet connections.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But from the IE implementers' perspective, partial evaluation is a step in the wrong direction.<span style="mso-spacerun: yes">  </span>ASP wants the compilation to go slow and the run to go fast because they are generating the code once, calling it a lot, and generating strings that must be served up as fast as possible.<span style="mso-spacerun: yes">  </span>IE wants the compilation to be as fast as possible because they want as little delay as possible between the HTML arriving over the network and the page rendering correctly.<span style="mso-spacerun: yes">  </span>They're never going to run the script again after its generated once, so there is no amortization of compilation cost.<span style="mso-spacerun: yes">  </span>And IE typically uses scripts to run user interface elements, not to build up huge strings as fast as possible.<span style="mso-spacerun: yes">  </span>Every microsecond does NOT count in most UI scenarios -- as long as the UI events are processed just slightly faster than we incredibly slow humans can notice the lag, everyone is happy. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

</div>

</div>

