<div id="page">

# Spot the Defect, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/16/2004 9:01:00 AM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Some fun for a Monday -- for some definition of fun, I suppose.  Here's a recent posting to Microsoft's internal **Spot The Defect** mailing list.</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">HRESULT CInvokeHelper::InvokeHelper(IDispatch \*pDisp, long dispid, SAFEARRAY \*\*param1)  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">{  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    HRESULT hr;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    DISPPARAMS params;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    EXCEPINFO hrInfo;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    VARIANTARG args\[1\];  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.cArgs=1;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.rgvarg=args;   
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.cNamedArgs=0;  
</span><span lang="NO" style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.rgvarg\[0\].vt=VT\_SAFEARRAY | VT\_I4;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    params.rgvarg\[0\].parray = \*param1;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    hr = pDisp-\>Invoke(dispid,IID\_NULL, LOCALE\_USER\_DEFAULT,   
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">       DISPATCH\_METHOD, \&params, NULL, \&hrInfo, NULL);  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    return returnVal(hr, hrInfo.scode);   
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">}  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">  
HRESULT CInvokeHelper::returnVal(HRESULT invokeHr, HRESULT scodeHR)  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">{  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    m\_invokeResult = invokeHr;          
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    if(FAILED(invokeHr))                    
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    {  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">        if(DISP\_E\_EXCEPTION == invokeHr)   
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">            return scodeHR;      
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">        return E\_FAIL;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    }  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">    return S\_OK;  
</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">}</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The internal Spot The Defect players found a good dozen or so defects -- some quite serious -- in this simple code.  How many can you find?</span>

</div>

</div>

