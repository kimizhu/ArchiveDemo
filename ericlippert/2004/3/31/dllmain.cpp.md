# dllmain.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2004 6:17:00 PM

-----

\#include "headers.h"

//  
// Helper methods  
//

static long g\_cReferences = 0;  
static long g\_cLocks = 0;  
static HMODULE g\_hmodule = NULL;

static const WCHAR \* pszEngineName      = L"SimpleScript";  
static const WCHAR \* pszOLEScript       = L"OLEScript";  
static const WCHAR \* pszCLSID           = L"CLSID";  
static const WCHAR \* pszProgID          = L"ProgID";  
static const WCHAR \* pszDescription     = L"SimpleScript Language Engine";  
static const WCHAR \* pszInprocServer    = L"InprocServer32";  
static const WCHAR \* pszActiveScripting = L"Active Scripting Engine";  
static const WCHAR \* pszParsing         = L"Active Scripting Engine with Parsing";  
static const WCHAR \* pszThreadingModel  = L"ThreadingModel";  
static const WCHAR \* pszBoth            = L"Both";  
static const WCHAR \* pszImplementedCats = L"Implemented Categories";

void DLLAddRef(void)  
{  
    InterlockedIncrement(\&g\_cReferences);  
}

void DLLRelease(void)  
{  
    InterlockedDecrement(\&g\_cReferences);  
}

void DLLAddLock(void)  
{  
    InterlockedIncrement(\&g\_cLocks);  
}

void DLLReleaseLock(void)  
{  
    InterlockedDecrement(\&g\_cLocks);  
}

static BOOL AttachProcess(HMODULE hmodule)  
{  
    g\_hmodule = hmodule;  
    return TRUE;  
}

static BOOL DetachProcess()  
{  
    return TRUE;  
}

static BOOL AttachThread()  
{  
    return TRUE;  
}

static BOOL DetachThread()  
{  
    return TRUE;  
}

static HRESULT OpenKey(HKEY hkeyParent, const WCHAR \* pszKey, HKEY \* phkey)  
{  
    Assert(hkeyParent \!= NULL);  
    AssertOutPtr(phkey);

    LONG err;  
    err = RegOpenKeyExW(hkeyParent, pszKey, 0, KEY\_ALL\_ACCESS, phkey);  
    return HRESULT\_FROM\_WIN32(err);  
}

static HRESULT CreateKey(HKEY hkeyParent, const WCHAR \* pszKey, HKEY \* phkey = NULL)  
{  
    Assert(hkeyParent \!= NULL);

    LONG err;  
    HKEY hkey = NULL;  
    HRESULT hr;

    err = RegCreateKeyExW(hkeyParent, pszKey, 0, NULL,  
        REG\_OPTION\_NON\_VOLATILE, KEY\_ALL\_ACCESS, NULL, \&hkey, NULL);  
    hr = HRESULT\_FROM\_WIN32(err);  
    if (FAILED(hr))  
        goto LError;

    if (phkey \!= NULL)  
    {  
        \*phkey = hkey;  
        hkey = NULL;  
    }

    hr = S\_OK;

LError:

    if (NULL \!= hkey)  
        RegCloseKey(hkey);

    return hr;  
}

static HRESULT SetValue(HKEY hkey, const WCHAR \* pszName, const WCHAR \* pszValue)  
{  
    Assert(hkey \!= NULL);  
    AssertReadStringN(pszName);  
    AssertReadString(pszValue);

    LONG err;

    err = RegSetValueExW(hkey, pszName, 0, REG\_SZ,  
        (const BYTE \*) pszValue, (wcslen(pszValue) + 1) \* sizeof WCHAR);

    return HRESULT\_FROM\_WIN32(err);  
}

//  
// Public entrypoints  
//

STDAPI DllGetClassObject(REFCLSID rclsid, REFIID riid, void \* \* ppv)  
{  
    HRESULT hr;  
    ClassFactory \* pFactory = NULL;

    if (NULL == ppv)  
        return E\_POINTER;

    AssertOutPtr(ppv);

    \*ppv = NULL;

    if (\!IsEqualCLSID(rclsid, CLSID\_SimpleScript))  
        return E\_INVALIDARG;

    hr = ClassFactory::Create(\&pFactory);  
    if (FAILED(hr))  
        goto LError;

    hr = pFactory-\>QueryInterface(riid, ppv);  
    if (FAILED(hr))  
        goto LError;

    hr = S\_OK;

LError:

    if (NULL \!= pFactory)  
        pFactory-\>Release();

    return hr;  
}

STDAPI DllUnregisterServer(void)  
{  
    // This method fails silently.

    HRESULT hr;  
     
    HKEY hkeyCLSID;  
    HKEY hkeyClassId;  
    HKEY hkeySimpleScript;  
    ICatRegister \* pCategoryRegister;

    const DWORD cchClassId = 39;  
    WCHAR pszClassId\[cchClassId\];  
    CATID catids\[2\];

    // HKCRCLSID  
    hr = OpenKey(HKEY\_CLASSES\_ROOT, pszCLSID, \&hkeyCLSID);  
    if (SUCCEEDED(hr))  
    {  
        StringFromGUID2(CLSID\_SimpleScript, pszClassId, cchClassId);  
        // HKCRCLSID{...}  
        hr = OpenKey(hkeyCLSID, pszClassId, \&hkeyClassId);  
        if (SUCCEEDED(hr))  
        {  
            hr = CoCreateInstance(CLSID\_StdComponentCategoriesMgr,  
                NULL, CLSCTX\_INPROC\_SERVER, IID\_ICatRegister, (void\*\*)\&pCategoryRegister);  
            if (SUCCEEDED(hr))  
            {  
                catids\[0\] = CATID\_ActiveScript;  
                catids\[1\] = CATID\_ActiveScriptParse;  
                pCategoryRegister-\>UnRegisterClassImplCategories(CLSID\_SimpleScript, 2, catids);  
                pCategoryRegister-\>Release();  
            }  
            RegDeleteKeyW(hkeyClassId, pszImplementedCats);  
            RegDeleteKeyW(hkeyClassId, pszOLEScript);  
            RegDeleteKeyW(hkeyClassId, pszProgID);  
            RegDeleteKeyW(hkeyClassId, pszInprocServer);  
            RegCloseKey(hkeyClassId);  
            RegDeleteKeyW(hkeyCLSID, pszClassId);  
        }  
        RegCloseKey(hkeyCLSID);  
    }

    hr = OpenKey(HKEY\_CLASSES\_ROOT, pszEngineName, \&hkeySimpleScript);  
    if (SUCCEEDED(hr))  
    {  
        RegDeleteKeyW(hkeySimpleScript, pszCLSID);  
        RegDeleteKeyW(hkeySimpleScript, pszOLEScript);  
        RegCloseKey(hkeySimpleScript);  
        RegDeleteKeyW(HKEY\_CLASSES\_ROOT, pszEngineName);  
    }

    return S\_OK;  
}

STDAPI DllRegisterServer(void)  
{  
    HRESULT hr;  
    DWORD error;

    HKEY hkeySimpleScript = NULL;  
    HKEY hkeyCLSID1 = NULL;  
    HKEY hkeyCLSID2 = NULL;  
    HKEY hkeyClassId = NULL;  
    HKEY hkeyProgID = NULL;  
    HKEY hkeyInprocServer = NULL;  
    ICatRegister \* pCategoryRegister = NULL;  
     
    CATEGORYINFO catinfo\[2\];

    const DWORD cchDllPathMax = MAX\_PATH + 1;  
    WCHAR pszDllPath\[cchDllPathMax\];  
    const DWORD cchClassId = 39;  
    WCHAR pszClassId\[cchClassId\];  
    DWORD cchDllPath;  
    CATID catids\[2\];  
     
    DllUnregisterServer(); // Ignore errors

    // HKCRSimpleScript  
    hr = CreateKey(HKEY\_CLASSES\_ROOT, pszEngineName, \&hkeySimpleScript);  
    if (FAILED(hr))  
        goto LError;

    // HKCRSimpleScriptOLEScript  
    hr = CreateKey(hkeySimpleScript, pszOLEScript);  
    if (FAILED(hr))  
        goto LError;

    // HKCRSimpleScript(Default) = "SimpleScript Language Engine"  
    hr = SetValue(hkeySimpleScript, NULL, pszDescription);  
    if (FAILED(hr))  
        goto LError;

    // HKCRSimpleScriptCLSID  
    hr = CreateKey(hkeySimpleScript, pszCLSID, \&hkeyCLSID1);  
    if (FAILED(hr))  
        goto LError;

    StringFromGUID2(CLSID\_SimpleScript, pszClassId, cchClassId);

    // HKCRSimpleScriptCLSID(Default) = "{...}"  
    hr = SetValue(hkeyCLSID1, NULL, pszClassId);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID  
    hr = CreateKey(HKEY\_CLASSES\_ROOT, pszCLSID, \&hkeyCLSID2);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}  
    hr = CreateKey(hkeyCLSID2, pszClassId, \&hkeyClassId);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}OLEScript  
    hr = CreateKey(hkeyClassId, pszOLEScript);  
    if (FAILED(hr))  
        goto LError;

    cchDllPath = GetModuleFileNameW(g\_hmodule, pszDllPath, cchDllPathMax - 1);  
    // GetModuleFileNameW doesn't necessarily terminate the buffer.  
    pszDllPath\[cchDllPathMax - 1\] = '  
    if (cchDllPath == 0)  
    {  
        error = GetLastError();  
        hr = HRESULT\_FROM\_WIN32(error);  
        if (FAILED(hr))  
            goto LError;  
    }

    // HKCRCLSID{...}(Default) = "SimpleScript Language Engine"  
    hr = SetValue(hkeyClassId, NULL, pszDescription);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}ProgID  
    hr = CreateKey(hkeyClassId, pszProgID, \&hkeyProgID);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}ProgID(Default) = "SimpleScript"  
    hr = SetValue(hkeyProgID, NULL, pszEngineName);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}InprocServer32  
    hr = CreateKey(hkeyClassId, pszInprocServer, \&hkeyInprocServer);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}InprocServer32(Default) = "c:simplescript.dll"  
    hr = SetValue(hkeyInprocServer, NULL, pszDllPath);  
    if (FAILED(hr))  
        goto LError;

    // HKCRCLSID{...}InprocServer32ThreadingModel = "Both"  
    hr = SetValue(hkeyInprocServer, pszThreadingModel, pszBoth);  
    if (FAILED(hr))  
        goto LError;

    hr = CoCreateInstance(CLSID\_StdComponentCategoriesMgr,  
        NULL, CLSCTX\_INPROC\_SERVER, IID\_ICatRegister, (void\*\*)\&pCategoryRegister);  
    if (FAILED(hr))  
        goto LError;

    catinfo\[0\].catid = CATID\_ActiveScript;  
    catinfo\[0\].lcid = MAKELANGID(LANG\_ENGLISH, SUBLANG\_ENGLISH\_US);  
    wcscpy(catinfo\[0\].szDescription, pszActiveScripting);  
    catinfo\[1\].catid = CATID\_ActiveScriptParse;  
    catinfo\[1\].lcid = MAKELANGID(LANG\_ENGLISH, SUBLANG\_ENGLISH\_US);  
    wcscpy(catinfo\[1\].szDescription, pszParsing);  
     
    hr = pCategoryRegister-\>RegisterCategories(2, catinfo);  
    if (FAILED(hr))  
        goto LError;

    catids\[0\] = CATID\_ActiveScript;  
    catids\[1\] = CATID\_ActiveScriptParse;

    hr = pCategoryRegister-\>RegisterClassImplCategories(CLSID\_SimpleScript, 2, catids);  
    if (FAILED(hr))  
        goto LError;

    hr = S\_OK;

LError:

    if (NULL \!= hkeyInprocServer)  
        RegCloseKey(hkeyInprocServer);  
    if (NULL \!= hkeyProgID)  
        RegCloseKey(hkeyProgID);  
    if (NULL \!= hkeyClassId)  
        RegCloseKey(hkeyClassId);  
    if (NULL \!= hkeyCLSID2)  
        RegCloseKey(hkeyCLSID2);  
    if (NULL \!= hkeyCLSID1)  
        RegCloseKey(hkeyCLSID1);  
    if (NULL \!= hkeySimpleScript)  
        RegCloseKey(hkeySimpleScript);  
    if (NULL \!= pCategoryRegister)  
        pCategoryRegister-\>Release();

    return hr;  
}

STDAPI DllCanUnloadNow (void)  
{  
    if (0 == g\_cReferences && 0 == g\_cLocks)  
        return S\_OK;  
    else  
        return S\_FALSE;  
}

EXTERN\_C BOOL WINAPI DllMain(HANDLE hmodule, DWORD dwReason, PVOID pvReserved)  
{  
    switch (dwReason)  
    {  
    case DLL\_PROCESS\_ATTACH:  
        return AttachProcess((HMODULE)hmodule);  
    case DLL\_THREAD\_ATTACH:  
        return AttachThread();  
    case DLL\_PROCESS\_DETACH:  
        return DetachProcess();  
    case DLL\_THREAD\_DETACH:  
        return DetachThread();  
    default:  
        Bug("DllMain() called with unrecognized dwReason.");  
        return FALSE;  
    }  
}

