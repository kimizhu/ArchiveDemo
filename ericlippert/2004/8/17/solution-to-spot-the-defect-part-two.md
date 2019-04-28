<div id="page">

# Solution to Spot the Defect Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/17/2004 11:35:00 AM

-----

<div id="content">

 There were l

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">ots of good responses to my challenge of yesterday.  There are two major defects in this code: that it leaks memory, and that it passes an incorrect variant type to Invoke.  There are also a considerable number of minor defects and questionable design decisions.  Let's go through that code a line at a time and see what could be better.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">HRESULT CInvokeHelper::InvokeHelper(IDispatch \*pDisp, long dispid, SAFEARRAY \*\*param1)  
{</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">1) There should be an assertion or runtime check that </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">pDisp</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is a pointer to good memory.  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">2) I would have typed the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">dispid</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> parameter as </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">DISPID</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, not </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">long</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  Same thing ultimately, but you know, we have these abstractions for a reason.  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">3) </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">\*param1</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is only ever read, not written.  There's no reason for it to be a </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">SAFEARRAY\*\*</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  It could be a </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">SAFEARRAY\*</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> just as well.  Couldn't hurt to put some assertions on there as well.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">non-defect: </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">param1</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> looks like a badly named parameter, but it actually makes some sense, as it will be the first (and only) parameter to the late-bound call.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    HRESULT hr;  
    DISPPARAMS params;  
</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">4) Defensive programming: </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">params</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> should be fully initialized, either with a </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">memset</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, or by setting the named arguments array to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">NULL</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    EXCEPINFO hrInfo;  
  
</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">5) </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">hrInfo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is a lousy name for an </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">EXCEPINFO</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> structure.  Why not </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">excepinfo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">?  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">6) Defensive programming: clear all the fields.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    VARIANTARG args\[1\];</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">non-defect: no need to initialize this with </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">VariantInit</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, as it's </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">vt</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is going to be set in just a moment.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.cArgs=1;  
    params.rgvarg=args;   
    params.cNamedArgs=0;  
    params.rgvarg\[0\].vt=VT\_SAFEARRAY | VT\_I4;</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">7) </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">VT\_SAFEARRAY</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> should be </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">VT\_ARRAY</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.rgvarg\[0\].parray = \*param1;  
    hr = pDisp-\>Invoke(dispid,IID\_NULL, LOCALE\_USER\_DEFAULT, </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">8) </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">LOCALE\_USER\_DEFAULT </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">isn't necessarily the best choice here.  Most callees ignore it, but there are potential gotchas.  I usually set it to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">0x0409</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, US-English.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">       DISPATCH\_METHOD, \&params, NULL, \&hrInfo, NULL);  
    return returnVal(hr, hrInfo.scode);   
  
</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">9) The callee is not guaranteed to set the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">scode</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> to anything -- </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">wcode</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> could be set instead, and then </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">scode</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> would be </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">S\_OK</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  Then the error info is lost\!  This thing should be passing in the whole </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">EXCEPINFO</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, not just one part of it.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">10) If the callee returned strings in the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">EXCEPINFO</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, they’ve just leaked memory.</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">}  
  
HRESULT CInvokeHelper::returnVal(HRESULT invokeHr, HRESULT scodeHR)  
{  
    m\_invokeResult = invokeHr;</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">11) OK, this is bizarre.  It stashes away the returned </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">HRESULT</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  What if the callee returned </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">DISP\_E\_EXCEPTION</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> and the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">EXCEPINFO</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> contained </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">E\_OUTOFMEMORY</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">?  What do we store?  </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">DISP\_E\_EXCEPTION</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, which tells you nothing.  Why is this being stored?  Hard to say without seeing the rest of the code, but it seems wrong.  
  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    if(FAILED(invokeHr))                    
    {  
        if(DISP\_E\_EXCEPTION == invokeHr)   
            return scodeHR;      
  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">As mentioned above, this could be </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">S\_OK</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> but there is still an error situation.    
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">  
        return E\_FAIL;  
  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">12) </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">E\_OUTOFMEMORY</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> comes in, </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">E\_FAIL</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> goes out, and if the error message ever gets to the user, they see "Something failed".  The error has been preserved in the class state of course, but who knows what it does with it.  Why lose the information in the first place?  It seems clear that the purpose of this method is to extract error information from an EXCEPINFO, but what it actually does is **<span style="FONT-WEIGHT: bold">changes</span>** how error information is stored and propagated **<span style="FONT-WEIGHT: bold">depending on where the error information came from</span>**.  That seems completely wrong -- you'd think you'd want to have a routine that makes how the error information is stored and propagated the same **<span style="FONT-WEIGHT: bold">no matter where</span>** the error information came from.  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">  
    }  
    return S\_OK;  
</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">  
13\) Again, this eats information.  The method might have returned S\_FALSE, but the caller will never know.  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">  
}</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">Clearly, testing error code paths is incredibly important.  They're easy to get wrong.</span>

</div>

</div>

