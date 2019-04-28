# Spot the Defect, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/16/2004 9:01:00 AM

-----

Some fun for a Monday -- for some definition of fun, I suppose.  Here's a recent posting to Microsoft's internal **Spot The Defect** mailing list.

HRESULT CInvokeHelper::InvokeHelper(IDispatch \*pDisp, long dispid, SAFEARRAY \*\*param1)  
{  
    HRESULT hr;  
    DISPPARAMS params;  
    EXCEPINFO hrInfo;  
    VARIANTARG args\[1\];  
    params.cArgs=1;  
    params.rgvarg=args;   
    params.cNamedArgs=0;  
    params.rgvarg\[0\].vt=VT\_SAFEARRAY | VT\_I4;  
    params.rgvarg\[0\].parray = \*param1;  
    hr = pDisp-\>Invoke(dispid,IID\_NULL, LOCALE\_USER\_DEFAULT,   
       DISPATCH\_METHOD, \&params, NULL, \&hrInfo, NULL);  
    return returnVal(hr, hrInfo.scode);   
}  
  
HRESULT CInvokeHelper::returnVal(HRESULT invokeHr, HRESULT scodeHR)  
{  
    m\_invokeResult = invokeHr;          
    if(FAILED(invokeHr))                    
    {  
        if(DISP\_E\_EXCEPTION == invokeHr)   
            return scodeHR;      
        return E\_FAIL;  
    }  
    return S\_OK;  
}

The internal Spot The Defect players found a good dozen or so defects -- some quite serious -- in this simple code.  How many can you find?

