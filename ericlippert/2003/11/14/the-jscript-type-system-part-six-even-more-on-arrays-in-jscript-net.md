<div id="page">

# The JScript Type System, Part Six: Even more on arrays in JScript .NET

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/14/2003 5:26:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">You might have noticed something odd about that last example using </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SetValue</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">. If you actually look up the method signature of </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SetValue</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> in the CLR documentation it notes that the function signature is:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-fareast-font-family: SimSun">public function SetValue(value : Object, indices : int\[\]) : void</span></span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">The odd thing is the annotation on </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">indices</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">. It is typed as taking a .NET array of integers but in the example in my last entry we give it a literal JScript array, not a hard-typed CLR array.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">JScript .NET arrays and hard-typed CLR arrays work together, but because these two kinds of arrays are so different they do not work together *perfectly*. The problem is essentially that JScript .NET arrays are much more dynamic than CLR arrays. JScript .NET arrays can change size, can have elements of any type, and so on. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">The rules for when JScript .NET arrays and CLR arrays may be used in place of each other are not particularly complicated but still you should exercise caution when doing so. In particular, when you use a JScript .NET array in a context where a CLR array is expected you can get unexpected results. Consider this example:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">function ChangeArray(arr : int\[\]) : void</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">{</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>print(arr\[0\]); // 10</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>arr\[0\] += 100;</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">}</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var jsarr : Array = new Array(10, 20, 30);</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">ChangeArray(jsarr);</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(jsarr\[0\]); // 10 or 110?</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">This might look like it prints out 10 then 110, but in fact it prints out 10 twice. **The compiler is unable to turn the dynamic JScript array into a hard-typed array of integers so it does the next best thing. It makes a copy of the JScript array and passes the copy to the function.** If the function reads the array then it gets the correct values. **If it writes it, then only the copy is updated, not the original.** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">To warn you about this possibly unintentional consequence of mixing array flavours, the compiler issues the following warning if you do that:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">warning JS1215: Converting a JScript Array to a System.Array results in a memory allocation and an array copy</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">You may now be wondering then why the call to </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SetValue</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> which had the literal JScript .NET array did not prompt this warning. **The warning is suppressed for literal arrays**. In the case of literal arrays the compiler can determine that a literal array is being assigned to a CLR array type. **The compiler then optimizes away the creation of the JScript .NET array and generates code to create and initialize the CLR array directly**. Since there is then no performance impact or unexpected copy, there is no need for a warning.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">Note that if every element of the source JScript .NET array cannot be coerced to the type of the hard-typed CLR array then a type mismatch error will be the result. For instance, if you tried to coerce an array containing strings to an array of integers then this would fail:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"> </span> </span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var arr1 : int\[\] = new Array(10, "hello", 20); // Type mismatch error at runtime</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var arr2 : int\[\] = \[10, "hello", 20\];<span style="mso-spacerun: yes">          </span>// Type mismatch error at compile time<span style="mso-spacerun: yes">  </span> </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">Note also that this applies to multidimensional arrays. **There is no syntax for initializing a multidimensional array in JScript .NET:** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var mdarr : int\[,\] = \[ \[ 1, 2 \], \[3, 4\] \]; // Nice try, but illegal</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">A **rectangular** multidimensional array is not an array of arrays. In this case you are assigning a one-dimensional array which happens to contain arrays to a two-dimensional array of integers. That is not a legal assignment. If, however, you want a **ragged** hard-typed array it is perfectly legal to do this:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var ragged : int\[\]\[\] = \[ \[ 1, 2 \], \[3, 4\] \];</span></span>

<span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(ragged\[1\]\[1\]); // 4</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">Rectangular multidimensional arrays are indexed with a comma-separated list inside one set of square brackets. If you use ragged arrays to simulate true multidimensional arrays then the indices each get their own set of brackets.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">Note that JScript .NET arrays do not have any of the methods or properties of a <span class="codeintext-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">CLR array</span></span>. (Strings, by contract, can be used implicitly as either JScript .NET strings or as </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">System.String</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> values, which I'll talk more about later.)<span style="mso-spacerun: yes">  </span>But JScript .NET arrays do not have <span class="codeintext-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">CLR array</span></span> fields like </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Rank</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">, </span><span class="codeintext-PRODUCTION"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SetValue</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">, and so on. <span style="mso-spacerun: yes"> </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: 0pt; mso-bidi-font-family: &#39;Times New Roman&#39;">Next time I'll talk a bit about going the other way -- using a CLR array where a JScript array is expected.</span>

</div>

</div>

