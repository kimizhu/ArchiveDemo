<div id="page">

# Let's Get Explicit\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/2/2003 1:57:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">A reader asked me yesterday if there was a way to detect "at compile time" (ie, before the code runs) whether a JScript program contained misspelled variables.<span style="mso-spacerun: yes">  </span>I'm sure we've all experienced the pain of </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var inv<span style="BACKGROUND: yellow; mso-highlight: yellow">ei</span>gledFroboznicator = new Froboznicator();</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">// ...</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(inv<span style="BACKGROUND: yellow; mso-highlight: yellow">ie</span>gledFroboznicator.frabness);</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The sooner bugs can be caught, the better, obviously.<span style="mso-spacerun: yes">  </span>We catch bugs like missing braces and unterminated strings before the script even runs, so why can't we catch use of undeclared identifiers?<span style="mso-spacerun: yes">  </span>Doesn't VBScript do that with </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Option Explicit</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Actually, no, it doesn't.<span style="mso-spacerun: yes">  </span>**Visual Basic** detects undeclared identifiers at compile time, but **VBScript** does not catch them until runtime.<span style="mso-spacerun: yes">  </span>The reason is because of the way the browser name lookup rules work.<span style="mso-spacerun: yes">  </span>Specifically:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* The </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">window</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> object is an **expando** object.<span style="mso-spacerun: yes">  </span>You can add new properties to it at runtime.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* All globally scoped variables and methods in a script block are automatically aggregated onto the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">window</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> expando.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">(Aside: a little known fact is that VBScript allows you to get around the second rule, though why you'd want to is beyond me. It is legal to put the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Private</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> keyword on a global declaration; such declarations will not be subsumed onto the window object.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So far there's nothing stopping us from finding undeclared identifiers at compile time, but then we add:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* The </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">window</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> object's top-level members are implicitly visible.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And now our dream of compile time analysis vanishes.<span style="mso-spacerun: yes">  </span>ANY identifier can be added to window at any time by another script block and therefore any identifier is potentially valid in every script block.<span style="mso-spacerun: yes">  </span>Add in the fact that the expando can be expanded with a string, and you end up with situations in which no amount of compile time analysis can possibly determine whether an identifier is legal or not.<span style="mso-spacerun: yes">  </span>Here's a really silly example.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">\<html\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">  </span>\<script language="jscript"\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">  </span><span style="mso-spacerun: yes">  </span>function dothething()</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>{</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">      </span>window\[NewVar\] = 123</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>}</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">  </span>\</script\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">  </span>\<script language="vbscript"\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>Option Explicit</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>Dim NewVar</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>NewVar = InputBox("Type in a word") </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>' type in "blah"</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>dothething()</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">    </span>window.alert blah</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes">  </span>\</script\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">\</html\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">This creates a new property on the window object which is not known until the user decides what to type.<span style="mso-spacerun: yes">  </span>Since the property is accessible without the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">window.</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> prefix, there's no way to know whether the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">blah</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> identifier is legal *until the program actually runs*. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Notice that JScript uses the variable declared in VBScript -- it is part of the window object because it is a global, in keeping with our rules laid out above.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Visual Basic does not allow access to top-level members of an object without qualification, so it knows that when it sees an undeclared variable, it really is a mistake.<span style="mso-spacerun: yes">  </span>(Which reminds me -- I wanted to tell you guys about ways to misuse the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">with</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> block, but that's another post.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I suppose that we could have added a feature to JScript like VBScript's </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Option Explicit</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, which catches this problem at run time, but we didn't.<span style="mso-spacerun: yes">  </span>As I mentioned on Sept 22nd, JScript already throws a runtime error when fetching an undeclared variable.<span style="mso-spacerun: yes">  </span>There is still a potential bug due to misspelling on a variable set, so be careful out there.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Also, as Peter reminded me, the JScript .NET compiler in compatibility mode can be used to detect things like undeclared variables, provided that you're willing to have cases like the one above show up as false positives.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">One more thing before I actually go do some real work: an interesting semantic difference between Visual Basic and VBScript is caused by the fact that the declaration check is put off until run time.<span style="mso-spacerun: yes">  </span>**Because an undeclared variable causes a run time error**, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">On Error Resume Next</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> **hides undeclared variables\!** So again, be careful out there\!<span style="mso-spacerun: yes">  </span>Don't rely on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Option Explicit</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> to save you next time you type with mittens on, because if you suppress errors, guess what?<span style="mso-spacerun: yes">  </span>Errors are suppressed\!</span>

</div>

</div>

