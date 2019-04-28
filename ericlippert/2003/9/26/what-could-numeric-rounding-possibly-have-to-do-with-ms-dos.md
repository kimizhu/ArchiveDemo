<div id="page">

# What could numeric rounding possibly have to do with MS-DOS?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/26/2003 7:10:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">A reader points out that </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatNumber</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> uses yet a different rounding algorithm.<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatNumber</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> rounds numbers ending in .5's away from zero, not towards evens. He also asks whether </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatNumber</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatCurrency</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, etc, actually call into the VBA </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Format$</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> code.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">To answer the question, no.<span style="mso-spacerun: yes">  </span>The VBA runtime is freely redistributable, but it is also **large**. <span style="mso-spacerun: yes"> </span>Back in 1996 we did not want to force people to download the large VBA runtime.<span style="mso-spacerun: yes">  </span>IE was on our case for every kilobyte we added to the IE download.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">However, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatNumber</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, etc, were written by the same developer who implemented </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Format$</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> in VBA.<span style="mso-spacerun: yes">  </span>That was the legendary Tim Paterson, who you may recall wrote a little program called QDOS that was eventually purchased by Microsoft and called MS-DOS.<span style="mso-spacerun: yes">  </span>And let me tell you, when I was an intern in 1993 I had to debug </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Format$ -- </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes">t</span>here are **very good reasons** why we decided to go with a stripped-down let's-solve-the-90%-usage-case version for VBScript\!<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Format$</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> is one of the most enormously complicated hard-core-bit-twiddling functions that I've ever seen.<span style="mso-spacerun: yes">  </span>That's what happens to these "everything but the kitchen-sink" functions that tries to be all things to all callers -- they get enormously bloated and complex.<span style="mso-spacerun: yes">  </span>Porting that thing to a new codebase and maintaining it independently would have been nightmarish.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Here's a comment that Tim wrote into the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FormatNumber</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> code on September 16th, 1996 which confirms the reader's observation:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">       </span>// We have more digits that we're not using.<span style="mso-spacerun: yes">    </span>Round if needed.</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">       </span>// This is simplistic rounding that does not use IEEE</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">       </span>// rules (rounding even if exactly x.5).<span style="mso-spacerun: yes">    </span>This simple</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">       </span>// way is how VBA Format$ works, so it's good enough for us.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

</div>

</div>

