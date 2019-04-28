<div id="page">

# Are JScript strings passed by reference?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2003 5:24:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Yesterday I asked "are JScript strings passed **by reference** (like JScript objects) or **by value** (like JScript numbers)?"</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Trick question\!<span style="mso-spacerun: yes">  </span>It doesn't matter, because you can't change a string.<span style="mso-spacerun: yes">  </span>I mean, suppose they were passed by reference -- how would you know?<span style="mso-spacerun: yes">  </span>You can't have two variables refer to the "same" string and then change that string.<span style="mso-spacerun: yes">  </span>Strings in JScript are like numbers -- immutable primitive values.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Of course, "under the covers" we actually have to pass the strings somehow.<span style="mso-spacerun: yes">  </span>Generally speaking, strings are passed by reference where possible, as it is much cheaper in both time and memory to pass a pointer to a string than to make a copy, pass the value, and then destroy the copy.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

</div>

</div>

