<div id="page">

# Compatibility vs. Performance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/24/2003 1:17:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Earlier I mentioned that two of the design goals for JScript .NET were **<span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">high performance</span></span>** and <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**compatibility with JScript Classic**</span></span>. Unfortunately these are somewhat contradictory goals\! JScript Classic has many dynamic features which make generation of efficient code difficult. Many of these features are rarely used in real-world programs. Others are programming idioms which make programs hard to follow, difficult to debug and slow.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">JScript .NET therefore has two modes: <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**compatibility mode**</span></span> and <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**fast mode**</span></span>. In compatibility mode there should be almost no JScript program which is not a legal JScript .NET program. Fast mode restricts the use of certain seldom-used features and thereby produces faster programs.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The JSC.EXE command-line compiler and ASP.NET both use fast mode by default. To turn fast mode off in JSC use the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">/fast-</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> switch. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Fast mode puts the following restrictions on JScript .NET programs:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\* **All variables must be declared with the** </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">var</span></span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> keyword.</span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> As I discussed earlier, in JScript Classic it is sometimes legal to use a variable without declaring it. In those situations, the JScript Classic engine automatically creates a new global variable but when in fast mode, JScript .NET does not. This is a good thing -- not only is the code faster but the compiler can now catch spelling errors in variable names.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\* **Functions may not be redefined**. In JScript Classic it is legal to have two or more identical function definitions which do different things. Only the **last** definition is actually used. This is not legal in JScript .NET in fast mode. This is also goodness, as it eliminates a source of confusion and bugs.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\* **Built-in objects are entirely read-only.** In JScript Classic it is legal to add, modify and (if you are perverse), delete some properties on the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Math</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> object, the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">String</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> prototype and the other built-in objects. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\* **Attempting to write to read-only properties now produces errors**. In JScript Classic writing to a read-only property fails silently, in keeping with the design principle I discussed earlier: muddle on through.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\* **Functions no longer have an** </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">arguments</span></span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> property.</span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> The primary use of the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">arguments</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> property is to create functions which take a variable number of arguments. JScript .NET has a specific syntax for creating such a function. This makes the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">arguments</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> object unnecessary. To create a JScript .NET function which takes any number of arguments the syntax is:</span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">function MyFunction(... args : Object\[\] )</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">{</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>// now use args.length, args\[0\], etc.</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">}</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Generally speaking, <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**unclear code is slow code**</span></span>. If the compiler is unable to generate good code it is usually because the restrictions on the objects described in the code are so loose as to make optimization impossible. These few restrictions not only let JScript .NET generate faster code, they also enforce good programming style without overly damaging the "scripty" nature of the language. And if you must run code which has undeclared variables, redefined functions, modified built-in objects or reflection on the function arguments, then there is always compatibility mode to fall back upon.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">JScript .NET also provides warnings when programming idioms could potentially produce slow code. For example, recall my earlier article on string concatenation.<span style="mso-spacerun: yes">  </span>Using the </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">+=</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> operator on strings now produces a warning which suggests using a </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">StringBuilder</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> instead. JScript .NET also produces warnings when code is likely to be incorrect. For example, using a variable before initializing it produces a warning.<span style="mso-spacerun: yes">  </span>So does branching out of a </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">finally</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> block now produce warnings, and so on.</span>

</div>

</div>

