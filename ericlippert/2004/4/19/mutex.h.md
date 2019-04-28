# mutex.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 6:22:00 PM

-----

\#ifndef MUTEX\_H // {  
\#define MUTEX\_H

class Mutex  
{  
     
private:

    CRITICAL\_SECTION m\_criticalsection;  
    BOOL m\_fInitialized;

    Mutex();

public:

    static HRESULT Create(Mutex \* \* ppMutex);  
    ~Mutex();  
    void Enter(void);  
    void Leave(void);  
};

\#endif // MUTEX\_H }

