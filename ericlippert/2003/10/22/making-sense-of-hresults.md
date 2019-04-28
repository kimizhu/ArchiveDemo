<div id="page">

# Making Sense of HRESULTS

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/22/2003 1:40:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Every now and then -- like, say, this morning -- someone sends me this mail:<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: blue; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">I'm getting an error in my JScript program.<span style="mso-spacerun: yes">  </span>The error number is -2147024877.<span style="mso-spacerun: yes">  </span>No description.<span style="mso-spacerun: yes">  </span>Help\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Making sense of those error numbers requires some delving into the depths of how COM represents errors -- the HRESULT.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">An HRESULT is a 32 bit unsigned integer where the high bit indicates whether it is an error or a success.<span style="mso-spacerun: yes">  </span>The remaining bits in the high word indicate the "facility" of the error -- into what broad category does this error fall?<span style="mso-spacerun: yes">  </span>The low word indicates the specific error for that facility.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">HRESULTS are therefore usually talked about in hex, as the bit structure is a lot easier to read in hex\!<span style="mso-spacerun: yes">  </span>Consider 0x80070013, for example.<span style="mso-spacerun: yes">  </span>The high bit is set, so this is an error.<span style="mso-spacerun: yes">  </span>The facility code is 7 and the error code is 0x0013 = 19 in decimal.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Unfortunately, JScript interprets the 32 bit error code as a **signed** integer and displays it in **decimal**.<span style="mso-spacerun: yes">  </span>No problem -- just convert that thing back to hex, right?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var x = -2147024877;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print(x.toString(16))</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Whoops, not quite.<span style="mso-spacerun: yes">  </span>JScript doesn't know that you want this as an unsigned number, so it converts it to a signed hex number, -0x7ff8ffed.<span style="mso-spacerun: yes">  </span>We need to convert this thing to the value it would have been had JScript interpreted it as an unsigned number in the first place.<span style="mso-spacerun: yes">  </span>A handy fact to know is that the difference between an unsigned number interpreted as a signed number and the same number interpreted as an unsigned number is always 0x100000000 if the high bit is set, 0 otherwise.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var x = -2147024877;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">print((x\<0?x+0x100000000:x).toString(16))</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">There we go.<span style="mso-spacerun: yes">  </span>That prints out 80070013.<span style="mso-spacerun: yes">  </span>Or, even better, we could just write a program that takes the error apart:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">function DumpHR(hr)</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">{</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>if (hr \< 0 )</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 2">            </span>hr += 0x100000000;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>if (hr & 0x80000000)</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 2">            </span>print("Error code");</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>else</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 2">            </span>print("Success code");</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>var facility = (hr & 0x7FFF0000) \>\> 16;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>print("Facility " + facility);</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>var scode = hr & 0x0000FFFF;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-tab-count: 1">      </span>print("SCode " + scode);</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">}</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">DumpHR(-2147024877);</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The facility codes are as follows</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_NULL<span style="mso-spacerun: yes">                    </span>0</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_RPC<span style="mso-spacerun: yes">                     </span>1</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_DISPATCH<span style="mso-spacerun: yes">                </span>2</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_STORAGE<span style="mso-spacerun: yes">                 </span>3</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_ITF<span style="mso-spacerun: yes">                     </span>4</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_WIN32<span style="mso-spacerun: yes">                   </span>7</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_WINDOWS<span style="mso-spacerun: yes">                 </span>8</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_SECURITY<span style="mso-spacerun: yes">                </span>9</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_CONTROL<span style="mso-spacerun: yes">                 </span>10</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_CERT<span style="mso-spacerun: yes">                    </span>11</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_INTERNET<span style="mso-spacerun: yes">                </span>12</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_MEDIASERVER<span style="mso-spacerun: yes">             </span>13</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_MSMQ<span style="mso-spacerun: yes">                    </span>14</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_SETUPAPI<span style="mso-spacerun: yes">                </span>15</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_SCARD<span style="mso-spacerun: yes">                   </span>16</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_COMPLUS<span style="mso-spacerun: yes">                 </span>17</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_AAF<span style="mso-spacerun: yes">                     </span>18</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_URT<span style="mso-spacerun: yes">                     </span>19</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_ACS<span style="mso-spacerun: yes">                     </span>20</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_DPLAY<span style="mso-spacerun: yes">                   </span>21</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_UMI<span style="mso-spacerun: yes">                     </span>22</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_SXS<span style="mso-spacerun: yes">      </span><span style="mso-spacerun: yes">               </span>23</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_WINDOWS\_CE<span style="mso-spacerun: yes">              </span>24</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_HTTP<span style="mso-spacerun: yes">                    </span>25</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_BACKGROUNDCOPY<span style="mso-spacerun: yes">          </span>32</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_CONFIGURATION<span style="mso-spacerun: yes">           </span>33</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_STATE\_MANAGEMENT<span style="mso-spacerun: yes">        </span>34</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">FACILITY\_METADIRECTORY<span style="mso-spacerun: yes">           </span>35</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So you can see that our example is a Windows operating system error (facility 7), and looking up error 19 we see that this is ERROR\_WRITE\_PROTECT -- someone is trying to write to a write-protected floppy probably.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">All the errors generated by the script engines -- syntax errors, for example -- are FACILITY\_CONTROL, and the error numbers vary between script engines.<span style="mso-spacerun: yes">  </span>VB also uses FACILITY\_CONTROL, but fortunately VBScript assigns the same meanings to the errors as VB does.<span style="mso-spacerun: yes">  </span>But in general, if you get a FACILITY\_CONTROL error you need to know what control generated the error -- VBScript, JScript, a third party control, what?<span style="mso-spacerun: yes">  </span>Because each control can define their own errors, and there may be collisions.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Finally, here are some commonly encountered HRESULTs:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_UNEXPECTED<span style="mso-spacerun: yes">  </span>0x8000FFFF</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> "Catestrophic failure" -- something completely unexpected has happened.</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_NOTIMPL<span style="mso-spacerun: yes">     </span>0x80004001</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> "Not implemented" -- the developer never got around to writing the method you just called\!</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_OUTOFMEMORY 0x8007000E</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- pretty obvious what happened here</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_INVALIDARG<span style="mso-spacerun: yes">  </span>0x80070057</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- you passed a bad argument to a method</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_NOINTERFACE 0x80004002</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- COM is asking an object for an interface.<span style="mso-spacerun: yes">  </span>This can happen if you try to script an object that doesn't support IDispatch.</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_ABORT<span style="mso-spacerun: yes">       </span>0x80004004</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- whatever you were doing was terminated </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">E\_FAIL<span style="mso-spacerun: yes">        </span>0x80004005</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- something failed and we don't know what.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And finally, here are three that you should see only rarely from script, but script hosts may see them moving around in memory and wonder what is going on:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SCRIPT\_E\_RECORDED<span style="mso-spacerun: yes">   </span>0x86664004</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- this is how we internally track whether the details of an error have been recorded in the error object or not.<span style="mso-spacerun: yes">  </span>We need a way to say "yes, there was an error, but do not attempt to record information about it again."</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SCRIPT\_E\_PROPAGATE<span style="mso-spacerun: yes">  </span>0x80020102</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- another internal code that we use to track the case where a recorded error is being propagated up the call stack to a waiting catch handler.</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">SCRIPT\_E\_REPORTED<span style="mso-spacerun: yes">   </span>0x80020101</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> -- the script engines return this to the host when there has been an unhandled error that the host has already been informed about via </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">OnScriptError</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">That's a pretty bare-bones look at error codes, but it should at least get you started next time you have a confusing error number.</span>

</div>

</div>

