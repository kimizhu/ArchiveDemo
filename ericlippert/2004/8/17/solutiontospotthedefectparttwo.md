# Solution to Spot the Defect Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/17/2004 11:35:00 AM

-----

 There were l

ots of good responses to my challenge of yesterday.  There are two major defects in this code: that it leaks memory, and that it passes an incorrect variant type to Invoke.  There are also a considerable number of minor defects and questionable design decisions.  Let's go through that code a line at a time and see what could be better.

HRESULT CInvokeHelper::InvokeHelper(IDispatch \*pDisp, long dispid, SAFEARRAY \*\*param1)  
{

1\) There should be an assertion or runtime check that pDisp is a pointer to good memory.  
2\) I would have typed the dispid parameter as DISPID, not long.  Same thing ultimately, but you know, we have these abstractions for a reason.  
3\) \*param1 is only ever read, not written.  There's no reason for it to be a SAFEARRAY\*\*.  It could be a SAFEARRAY\* just as well.  Couldn't hurt to put some assertions on there as well.

non-defect: param1 looks like a badly named parameter, but it actually makes some sense, as it will be the first (and only) parameter to the late-bound call.

    HRESULT hr;  
    DISPPARAMS params;  

4\) Defensive programming: params should be fully initialized, either with a memset, or by setting the named arguments array to NULL.

    EXCEPINFO hrInfo;  
  

5\) hrInfo is a lousy name for an EXCEPINFO structure.  Why not excepinfo?  
6\) Defensive programming: clear all the fields.

    VARIANTARG args\[1\];

non-defect: no need to initialize this with VariantInit, as it's vt is going to be set in just a moment.

    params.cArgs=1;  
    params.rgvarg=args;   
    params.cNamedArgs=0;  
    params.rgvarg\[0\].vt=VT\_SAFEARRAY | VT\_I4;

7\) VT\_SAFEARRAY should be VT\_ARRAY 

    params.rgvarg\[0\].parray = \*param1;  
    hr = pDisp-\>Invoke(dispid,IID\_NULL, LOCALE\_USER\_DEFAULT, 

8\) LOCALE\_USER\_DEFAULT isn't necessarily the best choice here.  Most callees ignore it, but there are potential gotchas.  I usually set it to 0x0409, US-English.

       DISPATCH\_METHOD, \&params, NULL, \&hrInfo, NULL);  
    return returnVal(hr, hrInfo.scode);   
  

9\) The callee is not guaranteed to set the scode to anything -- wcode could be set instead, and then scode would be S\_OK.  Then the error info is lost\!  This thing should be passing in the whole EXCEPINFO, not just one part of it.

10\) If the callee returned strings in the EXCEPINFO, they’ve just leaked memory.

}  
  
HRESULT CInvokeHelper::returnVal(HRESULT invokeHr, HRESULT scodeHR)  
{  
    m\_invokeResult = invokeHr;

11\) OK, this is bizarre.  It stashes away the returned HRESULT.  What if the callee returned DISP\_E\_EXCEPTION and the EXCEPINFO contained E\_OUTOFMEMORY?  What do we store?  DISP\_E\_EXCEPTION, which tells you nothing.  Why is this being stored?  Hard to say without seeing the rest of the code, but it seems wrong.  
  
    if(FAILED(invokeHr))                    
    {  
        if(DISP\_E\_EXCEPTION == invokeHr)   
            return scodeHR;      
  
As mentioned above, this could be S\_OK but there is still an error situation.    
  
        return E\_FAIL;  
  
12\) E\_OUTOFMEMORY comes in, E\_FAIL goes out, and if the error message ever gets to the user, they see "Something failed".  The error has been preserved in the class state of course, but who knows what it does with it.  Why lose the information in the first place?  It seems clear that the purpose of this method is to extract error information from an EXCEPINFO, but what it actually does is **changes** how error information is stored and propagated **depending on where the error information came from**.  That seems completely wrong -- you'd think you'd want to have a routine that makes how the error information is stored and propagated the same **no matter where** the error information came from.  
  
    }  
    return S\_OK;  
  
13\) Again, this eats information.  The method might have returned S\_FALSE, but the caller will never know.  
  
}

Clearly, testing error code paths is incredibly important.  They're easy to get wrong.

