# classfac.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2004 1:17:00 PM

-----

\#ifndef CLASSFAC\_H  
\#define CLASSFAC\_H

class ClassFactory : public IClassFactory  
{

private:  
         
    long m\_cref;

    ClassFactory();  
    ~ClassFactory();  
         
public:

    static HRESULT Create(ClassFactory \* \* ppFactory);

    // IUnknown  
    STDMETHODIMP QueryInterface(REFIID riid, void\*\* ppv);  
    STDMETHODIMP\_(ULONG) AddRef(void);  
    STDMETHODIMP\_(ULONG) Release(void);  
    // IClassFactory  
    STDMETHODIMP LockServer(BOOL fLock);  
    STDMETHODIMP CreateInstance(IUnknown \* punkOuter, REFIID riid, void \*\* ppv);  
};

\#endif // \!CLASSFAC\_H

