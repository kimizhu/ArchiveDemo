# JScript, DNA, and Mad Cow Disease

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2004 9:56:00 AM

-----

The [WebGraphics blog](http://web-graphics.com/)recently had a [Javascript quine contest](http://web-graphics.com/pages/quine.php) that just ended. I'm totally bummed that I didn't hear about this until it was over.  What's a "quine"?  A quine is a program which, when run, produces the source code of the program as its output.  The term is an homage to the late philosopher [Willard Van Orman Quine](http://www.wvquine.org/), and was coined by [Douglas Hofstadter](http://www.cogs.indiana.edu/people/homepages/hofstadter.html). There's really not much of a practical upshot to writing quines, but depending on the language you pick they can be quite challenging.  Here's an introductory paper on some of the [challenges in writing quines](http://www.cgl.uwaterloo.ca/~csk/washington/sdc_paper/toc.html) in C. One twist that actually makes this contest a little bit easier is that since JScript doesn't have a built-in "print out a string" function, the rules of the game state that to be a successful quine, the last expression evaluated by the program must be a string containing the program text.  This also seems "purer" to me; the JScript built-in functions are a documented, guaranteed-to-be-there part of the language, whereas printf and other C runtime library functions used to do the heavy lifting are not part of the language itself, merely commonly used standard runtime libraries. The submitted solutions are quite inventive, using everything from unescape to regular expressions, to reversed strings. Something that I find interesting about the submitted solutions to this problem (see the link above) is that though they use a number of clever and inventive techniques, they all basically come down to the same technique that a cell uses to reproduce. From a simplified, abstract reproductive standpoint a cell consists of two parts: the DNA **blueprint** which encodes instructions for building things out of proteins, and a **factory** which manufactures whatever the blueprint says. The cell makes a copy of the blueprint, builds whatever the blueprint says to build, and embeds the copy of the blueprint in the built structure. And since the blueprint in a cell describes how to build an identical cell, this process can continue on in the daughter cell. (If the blueprint does not describe how to build an identical cell, that might be because the cell has been infected by a virus which has hijacked the factory to make copies of the virus instead of the cell.  Or, perhaps the DNA has mutated, or the factory is damaged in some way.) Similarly, these programs all have a "DNA" **blueprint string** that describes the structure of something, they have a **universal constructor** which constructs a copy of the blueprint plus whatever the blueprint says to construct, and they have **a blueprint which describes the universal constructor itself**. Some of the constructors are more universal than others -- some really could construct any old string, some have been specifically tuned to solve the quine problem, but the basic idea is the same throughout. Here's the most straightforward and most universal example of what I mean: a='b="a=%27"+a+"%27;"+unescape(a)';b="a='"+a+"';"+unescape(a) Notice how the program clearly separates the static blueprint statement from the dynamic factory statement.  The blueprint just contains a pattern which the factory can use to build a string.  The factory produces a copy of the blueprint and then executes the blueprint to produce the result, and appends it to the blueprint.  Just like a cell produces a copy of the DNA and then builds a new cell to wrap around it.  This program could produce almost any output given a suitable blueprint; it just happens to have a blueprint for itself. To continue this analogy, I like to think of [cheater quines](http://www.cgl.uwaterloo.ca/~csk/washington/sdc_paper/discussion6.html) as **prions**.  Prions are the protein structures that cause Mad Cow Disease and related diseases. Unlike viruses and bacteria, prions contain no DNA blueprint whatsoever.  Rather, they replicate themselves in a really sneaky way.  A prion is simply a protein that is a variation on the shape of a "normal" protein often found in animal's brains.  But it is a very special shape -- it is one that has the property that when it touches a protein of the "normal" shape, it twists the normal protein into the prion shape, thereby replicating itself.  It's like if you had a special spoon of such a shape that whenever it touched a fork, the fork turned into a spoon. Throw that thing into a drawer full of forks, and pretty soon you'll have no forks left. The thing about prions is that they are completely "implementation detail dependent".  A cell is a **universal** constructor -- it can build almost **anything** out of proteins, from virii to rhinoceri.  You can change the structure of the blueprint and the factory will cheerfully build whatever the new blueprint specifies.  Prions can't build anything and do not contain instructions to build anything.  They're just shapes that happen to have the weird property that they can turn things that already sort of look like themselves into things that look exactly like themselves.  "Prion" quines are like that too -- they work by taking advantage of implementation details of the surrounding environment to reproduce. UPDATE: Here are two challenges.  Challenge \#1: What's the shortest JScript "pure" quine you can come up with?  That is, a quine that does not use *any* library functions.  (unescape, eval, regexp functions, charCodeAt, etc.) Here's my entry -- well, not exactly.  I've added carriage returns, spaces and comments to make it readable.  Without the CRs and comments it's a 240 character quine. // avoid having to escape quotes:  
var q='"';  
// DNA blueprint section  
var b=\[  
"var s=",  
q,  
"var q='",  
q,  
"+q+",  
q,  
"';var b=\[",  
q,  
";for(i in b)s+=b\[i\]==q?'q,':'",  
q,  
"'+b\[i\]+'",  
q,  
",';s+='\];';for(i in b)s+=b\[i\];",  
\];  
// reproduce the header  
var s="var q='"+q+"';var b=\[";  
// reproduce the blueprint  
for(i in b)  
  s+=b\[i\]==q ?  
    'q,' :  
    '"'+b\[i\]+'",';  
s+='\];';  
// run the factory on the blueprint  
for(i in b)  
  s+=b\[i\]; Challenge \#2: My program above, since it has those comments and CRs, is not a quite a quine. But it's last expression evaluated **is** a 240 character quine.  Let's call a program that is almost a quine in this manner a "quine uncle".  Can you find a JScript quine uncle which is SHORTER than the resulting quine?  How short can you make it?

