# Checking For Script Syntax Errors, This Time With Code

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/12/2005 12:37:00 PM

-----

A number of people asked me to clarify yesterday's entry. Rather than try to talk you through it, I think the code is straightforward enough to speak for itself. Here's a little skeleton that I just whipped up. \#include \<stdio.h\>  
\#include \<activscp.h\>  
\#include \<new\> const GUID CLSID\_VBScript = {0xb54f3741, 0x5b07, 0x11cf, {0xa4, 0xb0, 0x00, 0xaa, 0x00, 0x4a, 0x55, 0xe8}};  
const GUID CLSID\_JScript  = {0xf414c260, 0x6ac0, 0x11cf, {0xb6, 0xd1, 0x00, 0xaa, 0x00, 0xbb, 0xbb, 0x58}}; class MySite : public IActiveScriptSite {  
private:   ULONG m\_cref;  
  virtual ~MySite() {} public:   MySite() {  
    this-\>m\_cref = 1;  
  }   STDMETHOD\_(ULONG, AddRef)() {  
    return ++this-\>m\_cref;  
  }   STDMETHOD\_(ULONG,Release)() {  
    --this-\>m\_cref;  
    if (this-\>m\_cref == 0) {  
      delete this;  
      return 0;  
    }  
    return this-\>m\_cref;  
  }   STDMETHOD(QueryInterface)(REFIID iid, void \*\* ppv) {  
    if (ppv == NULL)  
      return E\_POINTER;  
    if (IsEqualIID(iid, IID\_IUnknown))  
      \*ppv = (IUnknown\*)this;  
    else if (IsEqualIID(iid, IID\_IActiveScriptSite))  
      \*ppv = (IActiveScriptSite\*)this;  
    else {  
      \*ppv = NULL;  
      return E\_NOINTERFACE;  
    }  
    this-\>AddRef();  
    return S\_OK;  
  }   STDMETHOD(GetLCID)(LCID \* plcid) {  
    return E\_NOTIMPL;  
  }   STDMETHOD(GetItemInfo)(  
    LPCOLESTR pstrName,  
    DWORD dwReturnMask,  
    IUnknown \*\* ppunkItem,  
    ITypeInfo \*\* ppti) {  
    return E\_NOTIMPL;  
  }   STDMETHOD(GetDocVersionString)(BSTR \* pbstrVersion) {  
    return E\_NOTIMPL;  
  }   STDMETHOD(OnScriptTerminate)(  
    const VARIANT \* pvarResult,  
    const EXCEPINFO \* pexcepinfo) {  
    return S\_OK;  
  }   STDMETHOD(OnStateChange)(SCRIPTSTATE state) {  
    return S\_OK;  
  }   STDMETHOD(OnEnterScript)() {  
    return S\_OK;  
  }  
   
  STDMETHOD(OnLeaveScript)() {  
    return S\_OK;  
  }   STDMETHOD(OnScriptError)(IActiveScriptError \* perror) {  
    EXCEPINFO excepinfo;  
    LONG column = 0;  
    ULONG line = 0;  
    DWORD context = 0;  
    BSTR bstrLine = NULL;  
    memset(\&excepinfo, 0x00, sizeof excepinfo);  
    perror-\>GetExceptionInfo(\&excepinfo);  
    if (excepinfo.pfnDeferredFillIn \!= NULL)  
      excepinfo.pfnDeferredFillIn(\&excepinfo);  
    perror-\>GetSourceLineText(\&bstrLine);  
    perror-\>GetSourcePosition(\&context, \&line, \&column);     wprintf(L"Error on line %ld column %ld\\n", line, column);  
    if (bstrLine \!= NULL)  
      wprintf(L"Line: %s\\n", bstrLine);  
    if (excepinfo.bstrDescription \!= NULL)  
      wprintf(L"Description: %s\\n", excepinfo.bstrDescription);  
    if (excepinfo.bstrSource \!= NULL)  
      wprintf(L"Source: %s\\n", excepinfo.bstrSource);  
    if (excepinfo.bstrHelpFile \!= NULL)  
      wprintf(L"Help: %s\\n", excepinfo.bstrHelpFile);     SysFreeString(bstrLine);  
    SysFreeString(excepinfo.bstrDescription);  
    SysFreeString(excepinfo.bstrSource);  
    SysFreeString(excepinfo.bstrHelpFile);     return S\_OK;  
  }  
}; void main() {  
  HRESULT hr = S\_OK;  
  HRESULT hrInit;  
  IClassFactory \* pfactory = NULL;  
  IActiveScript \* pscript = NULL;  
  IActiveScriptParse \* pparse = NULL;  
  IActiveScriptSite \* psite = NULL;   hr = hrInit = OleInitialize(NULL);  
  if (FAILED(hr))  
    goto LError;   hr = CoGetClassObject(CLSID\_VBScript, CLSCTX\_SERVER, NULL,  
    IID\_IClassFactory, (void\*\*)\&pfactory);  
  if (FAILED(hr))  
    goto LError;   hr = pfactory-\>CreateInstance(NULL, IID\_IActiveScript, (void\*\*)\&pscript);  
  if (FAILED(hr))  
    goto LError;   psite = new(std::nothrow) MySite();  
  if (psite == NULL) {  
    hr = E\_OUTOFMEMORY;  
    goto LError;  
  }   hr = pscript-\>SetScriptSite(psite);  
  if (FAILED(hr))  
    goto LError;   hr = pscript-\>QueryInterface(IID\_IActiveScriptParse, (void\*\*)\&pparse);  
  if (FAILED(hr))  
    goto LError;   hr = pparse-\>ParseScriptText(L"Function Foo \\n Foo = 123 \\n End Funtcion \\n",  
    NULL, NULL, NULL, 0, 1, 0, NULL, NULL);   if (FAILED(hr))  
    goto LError;  
   
LError:   if (FAILED(hr))  
    printf("%0x\\n", hr);   if (pparse \!= NULL)  
    pparse-\>Release();   if (psite \!= NULL)  
    psite-\>Release();   if (pscript \!= NULL) {  
    pscript-\>Close();  
    pscript-\>Release();  
  }   if (pfactory \!= NULL)  
    pfactory-\>Release();   if (SUCCEEDED(hrInit))  
    OleUninitialize();  
} As you would expect, this program prints out the information about the error, and ParseScriptText returns SCRIPT\_E\_REPORTED to indicate that there was an error but it has already been reported. Had there been no error, the script would not have actually run; the engine is not **started**, just **initialized**. Error on line 3 column 5  
Line:  End Funtcion  
Description: Expected 'Function'  
Source: Microsoft VBScript compilation error  
80020101

