<div id="page">

# The JScript Type System, Part Two: Prototypes and constructors

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/6/2003 5:01:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">A number of readers made some good comments on my article on JScript typing that deserve to be called out in more detail.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">First, I was being a little sloppy in my terminology -- I casually conflated **static typing** with **strong typing**, and **dynamic typing** with **weak typing**.<span style="mso-spacerun: yes">  </span>Thanks for calling me on that.<span style="mso-spacerun: yes">  </span>Under the definitions proposed by the reader, JScript would be a dynamically typed language (because every variable can take a value of any type) and a strongly typed language (because every object knows what type it is.)<span style="mso-spacerun: yes">  </span>By contrast, C++ is a statically typed language (because every variable must have a type, which the compiler enforces) but also a weakly typed language (because the reinterpret cast allows one to turn pointers into integers, and so on.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Second, a reader notes that one of the shortcomings of JScript is that though it is a strongly typed language (in our new sense) that **it is a royal pain to actually determine the runtime type an object**.<span style="mso-spacerun: yes">  </span>The </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">typeof</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> operator has a number of problems:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">null</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> is listed as being of the object type, though technically it is a member of the Null type. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* primitives (strings, numbers, Booleans) wrapped in objects are listed as being of the object type rather than their underlying type.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* JScript, unlike VBScript, does not interrogate COM objects to determine the class name.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">\* If JScript is passed a variant from the outside world that it cannot make sense of then </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">typeof</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> returns "unknown".</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Perhaps there is some other way.<span style="mso-spacerun: yes">  </span>Prototype inheritance affords a kind of type checking, for example.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Prototype inheritance works like this.  Every JScript object has an object (or possibly </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">null</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">) called its **prototype object**.  So suppose an object </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> has prototype object </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">bar</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">bar</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> has prototype object </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">baz</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">baz</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> has prototype object </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">null</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  If you call a method on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> then JScript will search </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">foo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">bar</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">baz</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> for that method, and call the first one it finds.<span style="mso-spacerun: yes">  </span>The idea is that one object is a prototypical object, and then other objects specialize it.<span style="mso-spacerun: yes">  </span>This allows for code re-use without losing the ability to dynamically customize behaviour of individual objects.</span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Prototypes are usually done something like this:</span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">var Animal = new Object();</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">// omitted: set up Animal object</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">function Giraffe(){ </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">// omitted: initialize giraffe object.</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">}</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Giraffe.prototype = Animal;</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">var Jerry = new Giraffe();</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Now </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Jerry</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> has all the properties and methods of an individual </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Giraffe</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> object AND all the properties and methods of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  You can use </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">IsPrototypeOf</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> to see if a given object has </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> on its prototype chain.<span style="mso-spacerun: yes">  </span>Since prototype chains are immutable once created, this gives you a pretty reliable sort of type checking.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Note that </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Giraffe</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is not a prototype of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Jerry</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.<span style="mso-spacerun: yes">  </span>Note also that </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is not the prototype of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Giraffe</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">\!<span style="mso-spacerun: yes">  </span>**The object which is assigned to the** </span>**<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> *<span class="underline">property</span>* of the constructor is the prototype of the instance.</span>**

 <span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Now, you guys are not the first people to point out to me that determining types is tricky.<span style="mso-spacerun: yes">  </span>A few years ago someone asked me what the differences are amongst </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">if (func.prototype.IsPrototypeOf(instance))</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">and</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">if (instance.constructor == func)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">and </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">if (instance instanceof func)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The obvious difference is that the first one looks at the **whole prototype chain**, whereas the second two look at **the** **constructor**, right? Or is that true?<span style="mso-spacerun: yes">  </span>Is there a semantic difference between the last two?<span style="mso-spacerun: yes">  </span>Actually, there is. </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Let's look at some examples, starting with one that seems to show that there is no difference:</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">function Car(){}</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var honda = new Car();</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(honda instanceof Car); // true</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(honda.constructor == Car);  // true</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">It appears that </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">instance instanceof func</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">instance.constructor == func</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> have the same semantics.   *They do not.*  Here's a more complicated example that demonstrates the difference:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var Animal = new Object();</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">function Reptile(){ }</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Reptile.prototype = Animal;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var lizard = new Reptile();</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(lizard instanceof Reptile); // true</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(lizard.constructor == Reptile); // false</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">In fact </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">lizard.constructor</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is equal to </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Object</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, not </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Reptile</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Let me repeat what I said above, because no one understands this the first time -- I didn't, and I've found plenty of Javascript books that get it wrong.<span style="mso-spacerun: yes">  </span>When we say</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Reptile.prototype = Animal;</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">this does NOT mean "the prototype of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Reptile</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">".  It cannot mean that because (obviously\!) the prototype of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Reptile</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, a function object, is </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Function.prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  No, this means "**the prototype of any instance of** </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Reptile</span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is </span>**<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">".  *There is no way to directly manipulate or read the prototype chain of an existing object.*</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Now that we've got that out of the way, the simple one first:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">instance instanceof func</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> means "is the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> <span style="mso-bidi-font-style: italic">property</span> of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">func</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> equal to any object on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">instance</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">'s prototype chain?"  So in our second example, the prototype property of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Reptile </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">is </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Animal </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">is on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">lizard</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">'s prototype chain.  </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But what about our first example where there was no explicit assignment to the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> prototype?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The compiler creates a function object called "</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">".  It **also creates a default prototype object and assigns it to** </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car.prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  So again, when we way </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(honda instanceof Car);</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">instanceof</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> operator gets the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> property (</span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car.prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">) and compares it to the prototype chain of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">honda</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  Since </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">honda</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> was constructed by </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> it gets </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Car.prototype</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> on its prototype chain.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">To sum up the story so far, <span style="mso-spacerun:"></span></span>

</div>

</div>

