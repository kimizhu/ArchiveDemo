<div id="page">

# Why Is There No \#Include?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/9/2003 1:48:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">A common and entirely sensible programming practice is to put commonly used utility functions in one file, and then somehow link that file in to many different programs.  In traditional compiled languages you can compile a bunch of utilities into a statically linked .LIB file or a dynamically linked .DLL file.  But what do you do in script, where a program is just text?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">People often ask me why there is no </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\#include</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> statement in VBScript or JScript.  Wouldn't it be nice to be able to write up a page full of helpful utility functions and then in half a dozen different scripts say  </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\#include "myUtilityFunctions.vbs"</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> ?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But if you actually think for a bit about how such a statement would be implemented, you see why we didn't do it.  The reason is pretty simple: **the script engine does not know where the current file came from, so it has no way of finding the referenced file**.  Consider IE, for example:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\<script language="vbscript"\></span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\#include "myUtilityFunctions.vbs"</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">' ...</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">How does the script engine know that this page was downloaded from <http://www.rezrov.com/gnusto/> ? The script engine knows nothing about where the script came from -- IE just hands the text to the engine.  Even assuming that it could figure it out, the script engine would then have to call IE's code to download files off the internet.  Worse, what if the path was fully qualified to a path in a different domain?  Now the script engine needs to implement IE's security rules.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But do any of those factors apply when running in ASP?  No\!  ASP would have to have a completely different </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\#include</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> mechanism where the relative path was based on facts about the current virtual root and the security semantics checked for cross-root violations.  What about WSH?  There we're worried about the current directory but have no security considerations.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">What about third party hosts?  **Every host could have a different rule for where the script comes from and what is a legal path**.  We cannot possibly put all those semantics into the script engine.  To make </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">an \#include</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> feature work, we'd have to define a callback function into the host so that the host could fetch the included script for us.  It's just too much work for too little gain, so we simply declared a rule: **the host is responsible for implementing any mechanism whereby script files can be included into the script namespace.**  This is why IE and WSH have the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\<script src=</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> attribute.</span>

</div>

</div>

